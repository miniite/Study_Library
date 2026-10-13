---
id: "TOOLS/MySQL/MySQL_Platform/Full-text_Search"
level: 4
type: "concept"
topic: "TOOLS/MySQL/MySQL_Platform"
tools: ["MySQL"]
---

# Full-text Search

**L4 Advanced** · [↑ MySQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A B-tree index can answer `LIKE 'term%'` (a prefix) but not `LIKE '%term%'`; for "contains this word" searches MySQL gives you `FULLTEXT` indexes with `MATCH … AGAINST`, and when you need ranking, typo tolerance, facets or scale, you graduate to a dedicated search engine fed from MySQL.

### 1. The big picture
Search features arrive in stages: first an exact lookup, then a "starts with" typeahead, then "contains", then ranking and typos. Each stage needs a different data structure. A B-tree stores keys in sorted order, so anything that can be expressed as a *range on the leftmost characters* is fast. Anything that depends on characters in the middle of a string cannot use sort order and falls back to scanning every row. An **inverted index** (word → list of rows) answers "which rows contain this word" directly, which is what `FULLTEXT` provides.

Related material in this library: *Grokking Relational Database Design* covers how indexes relate to keys and queries; here the focus is the MySQL implementation choices — `LIKE`, prefix indexes, `FULLTEXT`, and the point where Elasticsearch/OpenSearch is the right tool. The pack also includes the large-text column types (`TEXT` family) because they are what you usually search.

### 2. Core concepts and vocabulary
- **`LIKE`** — pattern match; `%` = any number of characters, `_` = exactly one.
- **Leading wildcard** — a pattern starting with `%` or `_`; makes a B-tree index useless.
- **Prefix index** — `INDEX (col(N))` indexes only the first N characters (required for `TEXT`/`BLOB`; optional for long `VARCHAR`).
- **Selectivity** — how well an index narrows rows: distinct values / total rows. A prefix is only useful if it is selective.
- **`FULLTEXT` index** — InnoDB/MyISAM inverted index on `CHAR`, `VARCHAR`, `TEXT` columns.
- **`MATCH(cols) AGAINST('terms' [mode])`** — the search predicate; also returns a relevance score.
- **Natural language mode** — default; ranks by relevance, no operators.
- **Boolean mode** — operators: `+word` required, `-word` excluded, `word*` prefix, `"exact phrase"`, `>`/`<` weight, `( )` grouping.
- **Query expansion** — `WITH QUERY EXPANSION` runs a second pass using terms from top results.
- **Stopwords / minimum token size** — words ignored by the index (`the`, `and`; tokens shorter than `innodb_ft_min_token_size`, default 3).
- **`TINYTEXT` / `TEXT` / `MEDIUMTEXT` / `LONGTEXT`** — 255 B / 64 KB / 16 MB / 4 GB length limits (plus a 1–4 byte length prefix).

### 3. How it works, step by step
**`LIKE` and the B-tree**
1. `WHERE name LIKE 'Mil%'` → optimizer converts to the range `name >= 'Mil' AND name < 'Mim'` and range-scans the index.
2. `WHERE name LIKE '%len'` or `'%ill%'` → no range exists; MySQL reads every row (or every index entry if the index covers the query) and tests each one.
3. Case/accent behaviour comes from the column collation (see the Charset lesson).

**`FULLTEXT`**
1. At index time the text is tokenized (split on non-word characters), stopwords and too-short tokens are dropped, and `(word → doc id, position)` entries are written to auxiliary tables.
2. `MATCH … AGAINST` tokenizes the query, looks up each word's posting list, intersects/unions them according to the mode, and computes a relevance score (term frequency × inverse document frequency in natural language mode).
3. The optimizer usually drives the plan from the FULLTEXT index (`type: fulltext` in `EXPLAIN`) and then joins/filters other predicates.

**Typeahead**
1. Debounce on the client (e.g., 150–250 ms) and require 2–3 characters.
2. Server runs `WHERE name LIKE CONCAT(?, '%') ORDER BY name LIMIT 10` on an indexed column.
3. Return only the 10 rows; never ship 50k options to the browser.

### 4. Worked example
```sql
CREATE TABLE article (
  id     BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  title  VARCHAR(200) NOT NULL,
  body   MEDIUMTEXT   NOT NULL,
  slug   VARCHAR(500) NOT NULL,
  FULLTEXT KEY ft_article (title, body),
  KEY ix_slug (slug(50))                   -- prefix index
) ENGINE=InnoDB;

-- Prefix search: uses ix_slug
EXPLAIN SELECT id FROM article WHERE slug LIKE 'how-to-%';

-- Contains search: full scan
EXPLAIN SELECT id FROM article WHERE body LIKE '%deadlock%';

-- Full-text, natural language: ranked
SELECT id, title, MATCH(title, body) AGAINST('deadlock retry') AS score
FROM   article
WHERE  MATCH(title, body) AGAINST('deadlock retry')
ORDER  BY score DESC
LIMIT  20;

-- Boolean mode: must contain "mysql", must not contain "oracle", words starting "lock"
SELECT id, title FROM article
WHERE MATCH(title, body) AGAINST('+mysql -oracle lock*' IN BOOLEAN MODE);

-- Phrase
SELECT id FROM article
WHERE MATCH(body) AGAINST('"gap lock"' IN BOOLEAN MODE);
```
Interpretation: the first plan shows `type: range` on `ix_slug`; the second `type: ALL` (or `index`). The `MATCH` queries show `type: fulltext`. The `score` column is a relative number (not a probability) and only comparable within one query.

**Sizing a prefix index**
```sql
SELECT COUNT(DISTINCT url)               / COUNT(*) AS full_selectivity,
       COUNT(DISTINCT LEFT(url, 10))     / COUNT(*) AS p10,
       COUNT(DISTINCT LEFT(url, 20))     / COUNT(*) AS p20,
       COUNT(DISTINCT LEFT(url, 40))     / COUNT(*) AS p40
FROM   pages;
```
Pick the smallest N whose selectivity is within a few percent of the full-column value. URLs that share `https://www.example.com/` need a longer prefix than names do.

### 5. Under the hood
- **Why leading wildcards fail.** B+tree order is lexicographic from the first character; knowing the middle of a string gives no starting point. An index that *covers* the query (all needed columns in the index) is still scanned fully, but without row lookups — faster than a table scan, still O(n).
- **InnoDB `FULLTEXT` layout.** Six auxiliary index tables (sharded by first letter/hash) hold the inverted index; a `FTS_DOC_ID` column (added implicitly or defined as `BIGINT UNSIGNED NOT NULL AUTO_INCREMENT` with a unique index named `FTS_DOC_ID_INDEX`) maps rows to postings. Inserted text first lands in an in-memory cache and is flushed at commit/threshold; deleted documents are tombstoned in `INNODB_FT_DELETED` until `OPTIMIZE TABLE` (with `innodb_optimize_fulltext_only`) purges them.
- **Visibility.** Matches are transactional: uncommitted rows aren't visible to other sessions' `MATCH` queries (unlike MyISAM).
- **Limits and knobs.** `innodb_ft_min_token_size` (3) and `innodb_ft_max_token_size` (84) are server-startup settings requiring an index rebuild; `innodb_ft_enable_stopword` and `innodb_ft_user_stopword_table`; `ngram` parser for CJK (`WITH PARSER ngram`). Boolean-mode `*` is only a trailing wildcard.
- **Relevance.** Natural-language scoring uses tf-idf-like weighting; InnoDB has no 50% common-word threshold (that was a MyISAM behaviour), but stopwords and small tables still produce surprising scores.
- **Collation.** Matching is case/accent behaviour of the column collation; no stemming, synonyms, or fuzzy matching.
- **TEXT storage.** Long values are stored off-page (overflow pages) under `DYNAMIC` row format, with only a 20-byte pointer in the row; reading a `TEXT` column costs extra page reads, so `SELECT *` on wide tables hurts and large objects (images, PDFs) belong in object storage with a URL in MySQL.

### 6. Performance and design implications
- **`LIKE 'x%'`**: index range scan, cost ∝ matches. **`LIKE '%x%'`**: cost ∝ table size.
- **`FULLTEXT`**: query cost ∝ posting-list lengths; very common words are expensive; combine with selective filters (`WHERE tenant_id = ? AND MATCH …`) — but know the optimizer will pick *one* driving index; sometimes it's better to run `MATCH` in a subquery/CTE and join.
- **Write cost.** Each insert/update re-tokenizes the changed text; bulk loads are faster if you load first and add the `FULLTEXT` index afterwards.
- **Prefix indexes** cut index size but can't be covering and can't enforce full-value uniqueness unless the prefix is enough.
- **Ranking and paging.** Deep `OFFSET` paging over scored results is O(offset); cap pages or use search-after tokens in a dedicated engine.
- **Typeahead**: small `LIMIT`, indexed prefix, cache hot prefixes; for substring typeahead consider a trigram/ngram helper table or a search engine.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `LIKE 'p%'`, `NOT LIKE`, `ESCAPE` | Pattern match | Prefix filters |
| `CREATE FULLTEXT INDEX … / ADD FULLTEXT` | Build inverted index | Word search |
| `MATCH(c1,c2) AGAINST('q' IN NATURAL LANGUAGE MODE / IN BOOLEAN MODE / WITH QUERY EXPANSION)` | Search | Ranked / operator search |
| `INDEX(col(N))` | Prefix index | Long strings, TEXT/BLOB |
| `innodb_ft_min_token_size`, `innodb_ft_enable_stopword` | Tokenization rules | Short tokens (e.g., 2-letter codes) |
| `WITH PARSER ngram`, `ngram_token_size` | CJK tokenization | East-Asian languages |
| `information_schema.INNODB_FT_*` | Inspect index words/cache/deleted | Debugging |
| `OPTIMIZE TABLE` | Rebuild; purge deleted FTS entries | Maintenance |
| `REGEXP` / `RLIKE` | Regular expressions | Small tables only; never sargable |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `LIKE 'prefix%'` + index | Substring or multi-word search |
| `FULLTEXT` — moderate data, simple word matching, want one datastore | Need typo tolerance, synonyms, stemming, facets, per-language analyzers |
| Prefix index | Prefix isn't selective (long common beginnings) |
| Dedicated engine (Elasticsearch/OpenSearch/Meilisearch/Typesense) — large corpora, relevance tuning, highlighting, aggregations | Team can't operate another cluster; eventual consistency is unacceptable |
| `TEXT` columns | Binary blobs (use object storage); frequent `SELECT *` |

**Migration path from `LIKE '%x%'`:** (1) add `FULLTEXT` and switch queries to `MATCH`; (2) validate relevance with real queries; (3) when requirements outgrow it, index into a search engine via an **outbox/CDC** pipeline (binlog → Kafka → indexer) rather than dual writes, keep MySQL as the source of truth, and design for re-indexing from scratch.

### 9. Common mistakes
- **`LIKE '%term%'` on large tables** → full scans → `FULLTEXT` or a search engine.
- **Expecting `MATCH` to find 2-letter words** → ignored by min token size → lower it and rebuild, or search another way.
- **Searching columns not covered by one `FULLTEXT` index** → `MATCH(a,b)` needs an index on exactly `(a,b)`.
- **Forgetting stopwords** → "the who" returns nothing → adjust stopword list.
- **Prefix index too short** → many rows share the prefix, index barely helps → measure selectivity.
- **Dual-writing to MySQL and Elasticsearch** → divergence on failure → outbox/CDC.
- **Wrapping the column in a function** (`LOWER(name) LIKE …`) → no index → rely on the collation or a generated column.
- **Loading 50k options into the UI** → janky page → server-side search with limits.

### 10. 🎯 Interview kit
**30-second answer:** B-tree indexes serve prefix searches (`LIKE 'abc%'`) but not contains searches. For word search use a `FULLTEXT` index and `MATCH … AGAINST` (natural language for ranking, boolean mode for operators). When you need ranking control, fuzzy matching, facets, or scale, move to Elasticsearch, fed from MySQL via CDC.

**Structure a longer answer:** query shapes → index structure fit → MySQL options (`LIKE`, prefix, `FULLTEXT`) with limits → decision criteria for a search engine → sync strategy and failure handling.

**Likely follow-ups:**
- *Natural language vs boolean mode?* → ranked relevance without operators vs operator-driven filtering with optional ranking.
- *How do you size a prefix index?* → compare `COUNT(DISTINCT LEFT(col,N))/COUNT(*)` to full selectivity.
- *Why can't `FULLTEXT` replace Elasticsearch?* → no analyzers/synonyms/fuzzy/facets/distributed scaling.
- *How do you keep the search index in sync?* → transactional outbox or binlog CDC, idempotent upserts, periodic reconciliation, full rebuild path.
- *`TEXT` vs `MEDIUMTEXT`?* → 64 KB vs 16 MB limit.

**Signals interviewers look for:** you tie each tool to a data-structure capability, you quantify (selectivity, limits), and you keep a single source of truth.

### 11. Check yourself
- [ ] Explain why `LIKE '%a'` can't use a B-tree index.
- [ ] Build a `FULLTEXT` index and write natural-language and boolean queries.
- [ ] Choose a prefix length from selectivity numbers.
- [ ] State two things `FULLTEXT` can't do that a search engine can.
- [ ] Sketch a CDC pipeline from MySQL to a search index and its failure modes.
- [ ] List the sizes of the four `TEXT` types.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../_index.md>)

## Practice questions

- [Basic] [What is the difference between TEXT, MEDIUMTEXT, and LONGTEXT and when would you use each?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33010_What_is_the_difference_between_TEXT_MEDIUMTEXT_and_LONGTEXT_.md>)
- [Intermediate] [A search feature uses LIKE 'term%' and is fast, but 'contains' search with LIKE '%term%' is slow — how do you explain the index difference and plan a fulltext migration?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33520_A_search_feature_uses_LIKE_term_and_is_fast_but_contains_sea.md>)
- [Intermediate] [When would you outgrow MySQL full-text search and move to a dedicated engine like Elasticsearch?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33078_When_would_you_outgrow_MySQL_full-text_search_and_move_to_a_.md>)
- [Intermediate] [How does the LIKE operator work and how do leading wildcards affect index usage?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33502_How_does_the_LIKE_operator_work_and_how_do_leading_wildcards.md>)
- [Intermediate] [A dropdown loads 50k options from MySQL and the page is janky — how do you split responsibility between server-side search endpoints and MySQL indexing for typeahead?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33314_A_dropdown_loads_50k_options_from_MySQL_and_the_page_is_jank.md>)
- [Deep] [Explain how you would size a prefix index on a long text column so it stays selective, and how you measure the selectivity trade-off.](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33466_Explain_how_you_would_size_a_prefix_index_on_a_long_text_col.md>)
- [Deep] [What is a prefix index and when would you use one on a long VARCHAR column?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33053_What_is_a_prefix_index_and_when_would_you_use_one_on_a_long_.md>)
- [Deep] [What is the difference between the MATCH ... AGAINST natural language mode and boolean mode in full-text search?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33077_What_is_the_difference_between_the_MATCH_AGAINST_natural_lan.md>)

## Related concepts

- [Leftmost Prefix Rule](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Leftmost_Prefix_Rule.md>) *(SQL Fundamentals)*
- [Index Access Paths](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Index_Access_Paths.md>) *(DBMS Concepts)*
- [Full-text Search in Postgres](<../../PostgreSQL/PostgreSQL_Platform/Full-text_Search_in_Postgres.md>) *(PostgreSQL Platform)*

## References

- *Grokking Relational Database Design* — Qiang Hao, Michail Tsikerdekis — Grokking Relational Database Design (cont.)
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — The Procedures › Chapter 6
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Creating a Full-Text Example
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Full-Text Search Index Catalog Views and Properties
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page Compression › LIKE Comparison
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 2: Data Types, NULLs and Conversions › Indexing and Storage Impact
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Advanced WHERE Clauses › Performing a Full-Text Search
- *Designing Data-Intensive Applications* — Martin Kleppmann — Storage and Retrieval › Full-text search and fuzzy indexes

