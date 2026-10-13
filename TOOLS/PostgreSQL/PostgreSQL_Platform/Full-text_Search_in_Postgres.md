---
id: "TOOLS/PostgreSQL/PostgreSQL_Platform/Full-text_Search_in_Postgres"
level: 4
type: "concept"
topic: "TOOLS/PostgreSQL/PostgreSQL_Platform"
tools: []
---

# Full-text Search in Postgres

**L4 Advanced** · [↑ PostgreSQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** PostgreSQL full-text search turns text into a `tsvector` of normalized lexemes, matches it against a `tsquery`, ranks results, and uses a GIN index to make the match fast — a good-enough search engine without leaving the database.

### 1. The big picture
`LIKE '%word%'` can't use a B-tree, ignores language (plural, stemming), and has no relevance ranking. Full-text search (FTS) solves all three: it tokenizes text, removes stop words, reduces words to stems ("running" → "run"), and indexes the result. It sits between plain `LIKE` / trigram fuzzy matching (`pg_trgm`) and a dedicated engine (Elasticsearch/OpenSearch). For moderate corpora where consistency with your relational data matters, Postgres FTS is often enough.

### 2. Core concepts and vocabulary
- **Document** — the text to be searched (one or more concatenated columns).
- **Lexeme** — a normalized token after parsing, stemming and stop-word removal.
- **`tsvector`** — sorted list of lexemes with positions (and optional weights A–D).
- **`tsquery`** — search expression of lexemes combined with `&` (and), `|` (or), `!` (not), `<->` (followed by).
- **Text search configuration** — language rules (`english`, `simple`, …): parser + dictionaries.
- **`@@`** — match operator: `tsvector @@ tsquery`.
- **Ranking** — `ts_rank`, `ts_rank_cd` order results by relevance.
- **GIN index** — inverted index lexeme → list of rows; the standard FTS index.
- **Generated column** — stored `tsvector` kept in sync automatically.

### 3. How it works, step by step
1. **Parse** the text into tokens (words, numbers, emails, URLs).
2. **Normalize** each through the configuration's dictionaries: lowercasing, stop-word removal, stemming.
3. **Store** as a `tsvector`, ideally in a generated column.
4. **Index** with GIN so lexeme lookups avoid scanning rows.
5. **Query**: build a `tsquery` (`plainto_tsquery`, `phraseto_tsquery`, `websearch_to_tsquery`, or `to_tsquery`), match with `@@`.
6. **Rank and highlight**: `ts_rank(vector, query)`, `ts_headline(text, query)`.

### 4. Worked example
```sql
CREATE TABLE articles (
  id     bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  title  text NOT NULL,
  body   text NOT NULL,
  search tsvector GENERATED ALWAYS AS (
           setweight(to_tsvector('english', coalesce(title, '')), 'A') ||
           setweight(to_tsvector('english', coalesce(body,  '')), 'B')
         ) STORED
);
CREATE INDEX articles_search_gin ON articles USING gin (search);

-- search like a user would type it
SELECT id, title,
       ts_rank(search, q) AS rank,
       ts_headline('english', body, q, 'MaxFragments=2') AS snippet
FROM articles, websearch_to_tsquery('english', 'postgres "full text" -mysql') AS q
WHERE search @@ q
ORDER BY rank DESC
LIMIT 20;

-- inspect what the engine does
SELECT to_tsvector('english', 'The runners were running quickly');
-- 'quick':5 'run':3 'runner':2   (positions kept, stop words dropped)
SELECT to_tsquery('english', 'run & quick');
```
Reading it: title matches (weight A) outrank body matches (B). `websearch_to_tsquery` accepts quoted phrases, `or`, and `-` negation without raising syntax errors, so it is the safest function for end-user input. Confirm index use with `EXPLAIN` — you should see a `Bitmap Index Scan on articles_search_gin`.

### 5. Under the hood   *(optional at this level)*
- A GIN index stores, per lexeme, a posting list/tree of row TIDs. Lookups are fast; **updates are slow** because one document touches many lexemes. GIN buffers inserts in a *pending list* (`fastupdate`, flushed by vacuum or when it exceeds `gin_pending_list_limit`), trading write latency for occasional flush spikes.
- GIST indexes are lossy and smaller, faster to update but slower to query; GIN is preferred for mostly-read corpora.
- The `english` configuration uses the Snowball stemmer; `simple` just lowercases (no stemming) — right for identifiers, names, or non-covered languages. Mixed-language data can use a per-row `regconfig` column.
- `ts_rank` considers term frequency and weights; `ts_rank_cd` also considers proximity (cover density). Neither is BM25; there's no inverse-document-frequency, so for sophisticated ranking you may need an external engine or extensions.
- Ranking requires fetching `tsvector`s for all matching rows, which is the slow part for very common words — add `LIMIT` and prefilter by other columns.

### 6. Performance and design implications
- Store the `tsvector` (generated column) instead of calling `to_tsvector()` in the query; an expression index also works but the expression in the query must be identical.
- Include only searchable columns; `tsvector` has a size limit (about 1 MB per value and positions ≤ 16383).
- For prefix search (autocomplete) use `to_tsquery('english', 'postg:*')`; for typo tolerance or substring match use `pg_trgm` with a GIN/GIST trigram index.
- Combine FTS with filters (`tenant_id = $1 AND search @@ q`); a composite approach with `btree_gin` or partitioning keeps big multi-tenant tables fast.
- Mind `gin_pending_list_limit` and `VACUUM` for write-heavy tables.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `to_tsvector(cfg, text)` | build document vector | indexing |
| `plainto_tsquery` / `websearch_to_tsquery` | user text → query | end-user input |
| `to_tsquery('a & b:*')` | operator query, prefix | programmatic search |
| `ts_rank`, `ts_rank_cd` | relevance score | ordering |
| `ts_headline` | highlighted snippet | result display |
| `setweight(vec, 'A')` | field importance | title vs body |
| `CREATE INDEX … USING gin` | inverted index | always for FTS |
| `default_text_search_config` | session default language | avoid passing config |
| `pg_trgm` + `%` / `similarity()` | fuzzy matching | typos, substrings |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Postgres FTS — data already in PG, modest scale, need transactional consistency | complex relevance (BM25), facets, synonyms at scale, heavy autocomplete |
| Generated stored `tsvector` + GIN | very write-heavy tables with huge documents |
| `pg_trgm` | long documents (index is large) |
| `simple` configuration | natural-language text where stemming helps |

### 9. Common mistakes
- Calling `to_tsvector(col)` without a config → depends on the session default, index not used → pass the config explicitly.
- Using `to_tsquery` on raw user input → syntax errors/injection of operators → `websearch_to_tsquery`.
- Expecting `LIKE`-style substring matching → FTS matches whole lexemes → use `:*` or trigrams.
- No index → seq scan computing vectors per row → add GIN.
- Mixing configs in index and query → no match → use the same `regconfig`.

### 10. 🎯 Interview kit
**30-second answer:** "Convert documents to `tsvector` (normalized lexemes), queries to `tsquery`, match with `@@`, rank with `ts_rank`, and speed it up with a GIN index on a stored generated column."
**Structure a longer answer:** why `LIKE` fails → pipeline → indexing → ranking → limits and alternatives.
**Likely follow-ups:** *GIN vs GiST?* → read speed vs update cost/size. *How to do autocomplete?* → prefix queries or trigrams. *Multi-language?* → per-row config. *When move to Elasticsearch?* → scale, relevance features.
**Signals interviewers look for:** you know it's an inverted index, you mention ranking and weights, and you're honest about its limits.

### 11. Check yourself
- [ ] Show what `to_tsvector('english', …)` produces for a sentence and why.
- [ ] Build a weighted generated column with a GIN index and prove the index is used.
- [ ] Explain why FTS updates are slower than B-tree updates and what `fastupdate` does.
- [ ] Decide between FTS and `pg_trgm` for a product-name search box.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is the difference between the = operator and the LIKE operator when matching text?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36137_What_is_the_difference_between_the_operator_and_the_LIKE_ope.md>)
- [Basic] [What is the LIKE operator and how does it differ from ILIKE?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36364_What_is_the_LIKE_operator_and_how_does_it_differ_from_ILIKE.md>)
- [Intermediate] [Your text search on a description column uses to_tsvector on every query and it's slow; how would you add a generated tsvector column with a GIN index and keep it in sync automatically?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36306_Your_text_search_on_a_description_column_uses_to_tsvector_on.md>)
- [Deep] [Your LIKE 'prefix%' query won't use the B-tree index in a non-C locale; explain why and how a text_pattern_ops operator class index fixes prefix matching under a locale-aware collation.](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36277_Your_LIKE_prefix_query_won_t_use_the_B-tree_index_in_a_non-C.md>)
- [Deep] [Users misspell search terms and full-text search returns nothing; how would you combine tsvector search with pg_trgm similarity for typo tolerance, and where would you draw the line and adopt a real search engine?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36309_Users_misspell_search_terms_and_full-text_search_returns_not.md>)
- [Deep] [Your pgvector similarity queries are accurate but slow at scale; how would you combine a metadata filter with an ANN index, and why can pre-filtering on a WHERE clause defeat the vector index?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36311_Your_pgvector_similarity_queries_are_accurate_but_slow_at_sc.md>)
- [Deep] [Your search feature uses ILIKE '%term%' and is unusably slow at scale; compare trigram (pg_trgm) indexes, full-text search, and moving to Elasticsearch and how you'd decide.](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35960_Your_search_feature_uses_ILIKE_term_and_is_unusably_slow_at_.md>)
- [Deep] [Your team wants full-text search but also fuzzy matching for typos; explain how you'd combine tsvector search with pg_trgm similarity and rank results, and where this stops being good enough.](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36080_Your_team_wants_full-text_search_but_also_fuzzy_matching_for.md>)

## Related concepts

- [Vector Similarity in SQL](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Vector_Similarity_in_SQL.md>) *(SQL Fundamentals)*
- [JSON and Text Search Indexes](<../../../01_Databases_and_SQL/Query_Performance/Indexing/JSON_and_Text_Search_Indexes.md>) *(Indexing)*
- [Full-text Search](<../../MySQL/MySQL_Platform/Full-text_Search.md>) *(MySQL Platform)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Optimizing LIKE Queries
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — P

