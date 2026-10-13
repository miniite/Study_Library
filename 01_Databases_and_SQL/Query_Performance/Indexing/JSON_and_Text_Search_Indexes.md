---
id: "01_Databases_and_SQL/Query_Performance/Indexing/JSON_and_Text_Search_Indexes"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Query_Performance/Indexing"
tools: ["PostgreSQL"]
---

# JSON and Text Search Indexes

**L4 Advanced** · [↑ Indexing](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** B-trees can't search *inside* a value, so JSON documents, arrays and free text need inverted-style indexes (GIN, trigram, full-text) or expression indexes on the specific keys you query — and each carries a real write and storage cost.

### 1. The big picture
A B-tree orders whole column values. That works for `id = 5` or `created_at > …`, but fails for questions about the *contents* of a value:
- "rows whose JSON has `status = 'paid'` somewhere inside"
- "rows whose `tags` array contains `'urgent'`"
- "rows whose text contains the word `refund`" or the substring `%abc%`

These need an **inverted index**: a map from each *element* (key/value pair, array element, word, trigram) to the list of rows that contain it. PostgreSQL implements this as **GIN** (Generalized Inverted Index); MySQL has full-text and multi-valued indexes; SQL Server has full-text and JSON via computed columns; search engines (Elasticsearch/OpenSearch/Pinot) are inverted-index systems at heart.

The practical decision is almost always: *which of these three do I use?*
1. A broad index over the whole document (flexible, expensive).
2. An **expression/partial index** on the one or two keys actually queried (cheap, specific).
3. **Promote** the key to a real column (best planner statistics, constraints, B-tree).

### 2. Core concepts and vocabulary
- **JSON vs JSONB (PostgreSQL)** — `json` stores text; `jsonb` stores a parsed binary form that supports indexing and fast key access.
- **Containment (`@>`)** — "document contains this sub-document"; the operator GIN's default JSONB opclass accelerates.
- **GIN index** — inverted index: entries are (element → posting list of TIDs). Fast reads for membership; slower, bursty writes (uses a pending list / fast update).
- **`jsonb_ops` vs `jsonb_path_ops`** — GIN opclasses: `jsonb_ops` indexes keys and values (supports `?`, `?|`, `?&`, `@>`); `jsonb_path_ops` indexes hashes of paths (smaller, faster, supports only `@>` and jsonpath).
- **Expression index** — B-tree on `(doc->>'status')` for equality/range on one key.
- **Generated/computed column** — stored value derived from JSON, indexable like a normal column.
- **Array operators** — `@>` (contains), `<@` (is contained by), `&&` (overlaps).
- **Full-text search** — `tsvector` (normalized lexemes) + `tsquery`; GIN-indexed.
- **Trigram (`pg_trgm`)** — 3-letter substrings; GIN/GiST makes `LIKE '%abc%'`, `ILIKE`, `%` similarity indexable.
- **Posting list** — the sorted list of row ids for a term.

### 3. How it works, step by step
GIN on a `jsonb` column `attrs`:
1. At insert, the engine extracts keys/values (or paths) from the document, e.g. `{"status":"paid","tags":["a","b"]}` → entries like `status`, `status:paid`, `tags`, `tags:a`, `tags:b`.
2. Each entry is added to the GIN structure (itself a B-tree of keys, each pointing to a posting list/tree of row TIDs).
3. For `WHERE attrs @> '{"status":"paid","tags":["a"]}'`, the engine looks up each required element, **intersects** the posting lists, and rechecks candidate rows against the actual document (the index is lossy for some opclasses).
4. Matching TIDs are fetched via a bitmap heap scan.

Contrast for a single key: `CREATE INDEX ON events ((attrs->>'status'))` builds a plain B-tree over the extracted string; `WHERE attrs->>'status' = 'paid'` seeks. It indexes **one** key, supports equality *and* range/sort, and is far smaller.

### 4. Worked example
```sql
-- PostgreSQL
CREATE TABLE events (id bigserial PRIMARY KEY, attrs jsonb NOT NULL, tags text[] NOT NULL DEFAULT '{}');

-- A) broad: any key/value containment
CREATE INDEX ix_events_attrs_gin ON events USING gin (attrs jsonb_path_ops);
SELECT count(*) FROM events WHERE attrs @> '{"status":"paid","country":"IN"}';

-- B) narrow: only two keys ever queried -> expression index (equality/range/order)
CREATE INDEX ix_events_status  ON events ((attrs->>'status'));
CREATE INDEX ix_events_amount  ON events (((attrs->>'amount')::numeric));
SELECT * FROM events WHERE attrs->>'status' = 'paid' ORDER BY (attrs->>'amount')::numeric DESC LIMIT 20;

-- C) promote: real column, populated by generated column
ALTER TABLE events ADD COLUMN status text GENERATED ALWAYS AS (attrs->>'status') STORED;
CREATE INDEX ix_events_status_col ON events (status);

-- Arrays
CREATE INDEX ix_events_tags ON events USING gin (tags);
SELECT * FROM events WHERE tags @> ARRAY['urgent','vip'];      -- has all
SELECT * FROM events WHERE tags && ARRAY['urgent','vip'];      -- has any

-- Substring search
CREATE EXTENSION IF NOT EXISTS pg_trgm;
CREATE INDEX ix_events_note_trgm ON events USING gin ((attrs->>'note') gin_trgm_ops);
SELECT * FROM events WHERE attrs->>'note' ILIKE '%refund%';
```
Verify each with `EXPLAIN (ANALYZE, BUFFERS)`: look for `Bitmap Index Scan on ix_events_attrs_gin` + `Bitmap Heap Scan` for GIN and `Index Scan` for expression indexes. Beware: the expression in the query must match the index expression exactly, including the cast `(attrs->>'amount')::numeric`. Queries using `->` (jsonb) vs `->>` (text) are different expressions.

### 5. Under the hood
- **Why GIN write cost is high.** One row insert can touch dozens of entries (every key/value/array element). PostgreSQL buffers into a *pending list* (`fastupdate`) and merges during vacuum or when the list exceeds `gin_pending_list_limit`, giving write bursts and occasional slow reads. Updating a JSONB document rewrites all its entries. On a table where 30 % of queries filter on JSONB and 70 % on scalar columns, a full GIN index can dominate the write budget: replace it with expression indexes on the few paths, or promote hot keys to columns.
- **When GIN stops helping.** Posting lists of very common elements are huge: `tags @> ARRAY['common']` matches most rows, so the planner correctly prefers a seq scan; GIN helps most when elements are selective. For "posts having all of tags A, B, C", GIN intersects lists (cheap when at least one is rare). Alternative: normalized `post_tags(post_id, tag_id)` with `GROUP BY … HAVING count(*) = 3` (good with a `(tag_id, post_id)` index and relational constraints).
- **Statistics.** The planner has no per-key statistics inside a JSONB blob (only a generic selectivity), so estimates are poor; a promoted column or expression index (which gets its own statistics after `ANALYZE`) fixes that.
- **Generated column rule.** Index a generated column when the expression is deterministic; stored generated columns persist the value (disk cost) but are visible to every tool and cast once. Virtual columns (MySQL, SQL Server non-persisted) are computed on read and can still be indexed in some engines. Maintainability: generated column = explicit schema, easy to reuse; expression index = no schema change but each query must restate the expression.
- **Full-text vs trigram.** Full-text works on *words* after stemming/stop-words; supports ranking (`ts_rank`); can't find substrings inside words. Trigram works on raw characters: supports `%abc%`, similarity and typos, but indexes are big and ranking is crude.
- **NoSQL analogy.** In DynamoDB/Cassandra-style stores, a *global secondary index* is a separate replicated table keyed on another attribute (eventually consistent, own capacity); a *local secondary index* shares the partition key (strongly consistent option, partition size limits). Covering/projection choices decide whether a query needs a second fetch.
- **Search tiers.** Elasticsearch, Pinot etc. choose among sorted, inverted, range, star-tree and bloom indexes per column — the same inverted-index idea with segment files, near-real-time refresh, and distribution.

### 6. Performance and design implications
- **Index the keys you query**, not the whole document; verify with usage statistics before dropping the broad index.
- **Prefer `jsonb_path_ops`** when you only need containment; `jsonb_ops` when you need key-existence operators.
- **Keep hot, filterable, joinable attributes as columns** — constraints, foreign keys, types and statistics are worth the schema change.
- **Bulk loads:** build GIN after loading; `fastupdate` tuning and `maintenance_work_mem`.
- **Combine** a scalar B-tree (tenant, date) with the GIN via bitmap AND, or put scalar columns in `btree_gin` multicolumn GIN.
- **Review a PR adding a 5-column composite:** ask for the target query, whether a prefix exists, write rate, size, benchmark on production-shaped data, and plan evidence.
- **Boundary with search platforms:** keep exact filters, small-scale FTS and trigram in Postgres while the requirement is "good enough" and transactional; move to a search tier for relevance ranking, synonyms, language analysers, facets, or very large corpora — and budget the sync pipeline and reconciliation.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `USING gin (col jsonb_path_ops)` | Compact containment index | `@>` on JSONB |
| `((col->>'key'))` expression index | B-tree on one key | Equality/range/order on one key |
| `GENERATED ALWAYS AS (…) STORED` | Materialised column | Reused attribute, stats |
| `USING gin (array_col)` | Array membership | `@>`, `&&` |
| `pg_trgm` + `gin_trgm_ops` | Substring/fuzzy index | `LIKE '%x%'`, `similarity()` |
| `to_tsvector`, `to_tsquery`, `ts_rank` | Full-text search | Word search with ranking |
| `gin_pending_list_limit`, `fastupdate` | GIN write buffering | Write-heavy GIN tuning |
| `jsonb_path_query`, `@@` | JSONPath | Complex JSON predicates (use path index) |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Expression index: 1–3 known keys | Many unpredictable keys |
| Full GIN: ad-hoc containment over varied keys | Write-heavy table, huge documents |
| Promote to column: key is hot/joined/constrained | Schema churn is the point of using JSON |
| Trigram: substring and typo-tolerant | Corpus is huge and ranking matters |
| Search tier: relevance, facets, scale | You need immediate consistency and have a small team |

### 9. Common mistakes
- **Putting a B-tree on a JSONB column** → only whole-document equality works → use GIN or expression indexes.
- **Query uses `->` but index is on `->>`** → seq scan → match the expression exactly, including casts.
- **Full GIN on a write-heavy table** → write latency and bloat → narrow indexes or promote keys.
- **Expecting GIN to help for very common tags** → planner picks seq scan → partial index on rare tags, or restructure.
- **Using `LIKE '%x%'` without trigram** → scan → `pg_trgm`.
- **Storing everything as JSON** → no constraints, poor stats → relational columns for core attributes.
- **Dual-writing to a search engine without reconciliation** → drift → CDC/outbox + periodic rebuild.

### 10. 🎯 Interview kit
**30-second answer:** B-trees index whole values, so searching inside JSON, arrays or text needs an inverted index like PostgreSQL's GIN, or an expression index on the specific key. I choose the narrowest option that serves the queries: expression index for a few keys, GIN for ad-hoc containment, a real column for hot attributes, trigram or full-text for text.

**Structure a longer answer:** why B-tree fails → inverted index mechanism → three options for JSONB with write/space cost → arrays and text → statistics and planner implications → when to leave the database for a search service.

**Likely follow-ups:**
- *Only two keys are queried — what do you do?* → Expression (or partial) indexes, or generated columns; full GIN is wasteful.
- *Array `@>` index stops helping when?* → Elements very common (low selectivity) → seq scan better; maybe partial index or normalized table.
- *Generated column vs expression index?* → Generated: visible, reusable, stats; expression: no schema change.
- *Trigram vs full-text?* → Substring/typo vs word/rank.
- *Postgres vs Elasticsearch?* → Transactional, simple, small corpus vs relevance, scale, analysis; define the boundary and sync strategy.

**Signals interviewers look for:** correct operator/opclass knowledge, write-cost awareness, pragmatic escalation path.

### 11. Check yourself
- [ ] Explain GIN in terms of posting lists and how a containment query uses them.
- [ ] Choose among GIN, expression index and promoted column for a table with two queried JSON keys.
- [ ] Write an index for `WHERE tags && ARRAY['x']`.
- [ ] Explain why `ILIKE '%abc%'` needs `pg_trgm`.
- [ ] Describe what could make a GIN index slow to write and how to mitigate it.
<!-- explainer:end -->

## Implementations (tools)

- [PostgreSQL](<../../../TOOLS/PostgreSQL/_index.md>)

## Practice questions

- [Intermediate] [How would you review a teammate's PR that adds a five-column composite index? What questions do you ask about the query it targets before approving?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19686_How_would_you_review_a_teammate_s_PR_that_adds_a_five-column.md>)
- [Deep] [How would you index and query a Postgres array column for "contains element X" lookups?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19624_How_would_you_index_and_query_a_Postgres_array_column_for_co.md>)
- [Deep] [How do you decide whether a query filtering on a JSONB field should get an expression index on that key, a full GIN index, or a promoted real column?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20128_How_do_you_decide_whether_a_query_filtering_on_a_JSONB_field.md>)
- [Deep] [A query filters WHERE tags @> ARRAY'x','y' on a Postgres array column. Explain how a GIN index serves it and when it stops helping (e.g. very common tags).](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19733_A_query_filters_WHERE_tags_ARRAY_x_y_on_a_Postgres_array_col.md>)
- [Deep] [You own a table where 30% of queries filter on a JSONB path and 70% on scalar columns. How do you avoid a heavy GIN index dominating your write budget?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19776_You_own_a_table_where_30_of_queries_filter_on_a_JSONB_path_a.md>)
- [Deep] [A many-to-many tag search 'posts having all of tags A, B, C' is slow. Compare a GIN index on a tag array, a normalized join with grouping, and an inverted-index table.](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19852_A_many-to-many_tag_search_posts_having_all_of_tags_A_B_C_is_.md>)
- [Expert] [How would you efficiently search a JSONB column in PostgreSQL?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19525_How_would_you_efficiently_search_a_JSONB_column_in_PostgreSQ.md>)
- [Expert] [In Apache Pinot, how do you choose among sorted, inverted, star-tree, and range indexes for a given query pattern in a real-time analytics table?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20420_In_Apache_Pinot_how_do_you_choose_among_sorted_inverted_star.md>)

## Related concepts

- [Indexing Multi-join Queries](<../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Indexing_Multi-join_Queries.md>) *(PostgreSQL Platform)*
- [Index Access Paths](<../../Database_Internals/DBMS_Concepts/Index_Access_Paths.md>) *(DBMS Concepts)*
- [Large Values and TOAST](<../../SQL_Language/SQL_Fundamentals/Large_Values_and_TOAST.md>) *(SQL Fundamentals)*

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 12. The Future of Data Systems › Combining Specialized Tools by Deriving Data
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Infrastructure Operations › Indexing

