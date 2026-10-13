---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between the LIKE operator and full-text search?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Leftmost Prefix Rule](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Leftmost_Prefix_Rule.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question asks for the difference between the LIKE operator and full-text search (FTS). At the "intermediate experience" level for a backend/fullstack SDE-2/3, the interviewer is checking whether you know these are fundamentally different tools: LIKE is substring/pattern matching on raw character strings, while FTS is linguistic, token-based relevance search with an inverted index. They want you to articulate the differences in matching semantics, performance/indexing, relevance ranking, and when to use which — and ideally to name the real engine features (Postgres tsvector/tsquery, MySQL FULLTEXT/MATCH ... AGAINST, and where a dedicated engine like Elasticsearch enters).

Clarifying questions worth voicing: Which database? Are we matching exact substrings (like SKUs/prefixes) or searching natural-language text (articles, product descriptions)? Do we need ranking, stemming, or typo tolerance? Data volume?

What LIKE actually does

LIKE performs pattern matching on the literal character sequence using two wildcards: % (any run of characters) and _ (a single character).

SQL
SELECT * FROM articles WHERE title LIKE '%database%';

Characteristics:

Substring, not word-aware. '%cat%' matches "cat", "category", and "concatenate" — it has no idea what a word is.
No linguistics. No stemming ("running" ≠ "run"), no stop-word removal, no synonyms, and (by default) it's case-sensitive in many collations (ILIKE in Postgres for case-insensitive).
Index usage is limited. A leading wildcard ('%term%' or '%term') cannot use a normal B-tree index → full table scan, O(n) per query, which degrades badly at scale. Only a prefix pattern ('term%') can use a B-tree index. (Postgres can accelerate arbitrary LIKE with a pg_trgm GIN/GiST trigram index — a useful nuance to mention.)
No relevance ranking. Rows either match or they don't; there's no notion of "best match first."

So LIKE is perfect for exact prefixes, codes, patterns, and small tables — cheap and simple.

What full-text search does

FTS treats text as language. It tokenizes text into terms, normalizes them (lowercasing, stemming so "running/ran/runs" → "run"), drops stop words ("the", "a"), and stores the terms in an inverted index (term → list of documents/positions). A query is likewise tokenized and matched against that index, then results are ranked by relevance (e.g., TF-IDF / BM25).

PostgreSQL:

SQL
-- one-off
SELECT * FROM articles
WHERE to_tsvector('english', body) @@ plainto_tsquery('english', 'running databases')
ORDER BY ts_rank(to_tsvector('english', body),
                 plainto_tsquery('english', 'running databases')) DESC;

-- indexed: store a tsvector column with a GIN index
ALTER TABLE articles ADD COLUMN doc tsvector
  GENERATED ALWAYS AS (to_tsvector('english', title || ' ' || body)) STORED;
CREATE INDEX idx_articles_doc ON articles USING GIN (doc);

MySQL / InnoDB:

SQL
CREATE FULLTEXT INDEX ft_body ON articles (title, body);
SELECT *, MATCH(title, body) AGAINST('running databases' IN NATURAL LANGUAGE MODE) AS score
FROM articles
WHERE MATCH(title, body) AGAINST('running databases' IN NATURAL LANGUAGE MODE)
ORDER BY score DESC;

What FTS gives you that LIKE can't:

Word-boundary matching — searching "run" matches the word "run", not "runner" or "overrun" (unless stemmed), avoiding LIKE's false positives.
Stemming & stop words — "databases" matches "database"; noise words ignored.
Boolean / phrase / proximity operators — AND/OR/NOT, "exact phrase", prefix (data:*).
Relevance ranking — order results best-first (ts_rank, MATCH ... score, BM25 in Elasticsearch).
Sub-linear, index-backed lookups at scale via the inverted index.

Its costs: the inverted index takes space and must be maintained on writes; setup is more involved; and it's tuned to a language (the wrong language config gives poor stemming).

When to use which (the real decision)
Use LIKE for: exact substrings and prefixes, SKUs/codes/emails, small tables, starts-with autocomplete on an indexed prefix, patterns with _/%. Simple, no extra index, exact control.
Use FTS for: natural-language content — articles, comments, product descriptions, documents — where you want word-aware, ranked, linguistically smart search over large volumes.
Reach beyond the DB (Elasticsearch/OpenSearch/Meilisearch) when you need: typo tolerance / fuzzy matching, complex relevance tuning (boosting fields, custom analyzers), faceting/aggregations, autocomplete-as-you-type, synonyms at scale, and search across huge corpora. Postgres FTS is excellent up to moderate scale and keeps search transactional with your data; a dedicated engine wins on advanced relevance, fuzzy search, and horizontal scale — at the cost of running and syncing a second datastore. LIKE and DB-native FTS notably do not do typo tolerance out of the box (that's trigram similarity or a search engine).
Summary / what a strong answer emphasizes

LIKE = character-level pattern matching: no language awareness, poor with leading wildcards (table scan), no ranking — great for exact substrings/prefixes and small data. Full-text search = token-based linguistic search over an inverted index: tokenization, stemming, stop words, boolean/phrase queries, and relevance ranking, scaling to large text corpora. Choose LIKE for precise pattern/prefix needs; choose FTS for natural-language relevance search; move to a dedicated search engine (Elasticsearch) when you need fuzzy/typo tolerance, advanced ranking, or big scale.

#### Common mistakes
using LIKE '%term%' for search on a big table (silent full scans), assuming LIKE matches whole words, and assuming DB FTS gives typo tolerance (it doesn't — that needs trigrams or a search engine). A good candidate also mentions pg_trgm as the bridge that makes LIKE/similarity indexable in Postgres.

**Comparison:**

```
Aspect	LIKE	Full-text search
Matching unit	raw character substring	linguistic tokens (words)
Word awareness	none (%cat% hits "category")	yes (word boundaries)
Stemming / stop words	no	yes ("running" -> "run", drops "the")
Boolean / phrase / proximity	no (only %, _)	yes (AND/OR/NOT, "phrase", prefix)
Relevance ranking	no	yes (ts_rank, BM25, MATCH score)
Index used	B-tree only for prefix ('x%'); scan for '%x%'	inverted index (GIN / FULLTEXT)
Scale on large text	poor (table scans)	good (sub-linear)
Typo / fuzzy tolerance	no	not by default (needs trigram / search engine)
Setup cost	none	index + config (language)
Best for	exact substrings, prefixes, codes, small tables	natural-language relevance search
Need	Tool
Prefix / code / exact substring	LIKE (with B-tree or pg_trgm)
Ranked natural-language search in-DB	Postgres tsvector / MySQL FULLTEXT
Typo tolerance, faceting, big scale, custom relevance	Elasticsearch / OpenSearch / Meilisearch
```

**Code:**

```sql
-- LIKE: substring pattern match (leading % -> no B-tree index -> table scan)
SELECT * FROM articles WHERE title LIKE '%database%';       -- scan
SELECT * FROM articles WHERE sku   LIKE 'AB-%';             -- prefix, can use B-tree

-- Postgres: make arbitrary LIKE fast with a trigram index
CREATE EXTENSION IF NOT EXISTS pg_trgm;
CREATE INDEX idx_title_trgm ON articles USING GIN (title gin_trgm_ops);

-- Postgres full-text search with a stored tsvector + GIN index and ranking
ALTER TABLE articles ADD COLUMN doc tsvector
  GENERATED ALWAYS AS (to_tsvector('english', title || ' ' || body)) STORED;
CREATE INDEX idx_articles_doc ON articles USING GIN (doc);

SELECT id, title,
       ts_rank(doc, plainto_tsquery('english', 'running databases')) AS rank
FROM articles
WHERE doc @@ plainto_tsquery('english', 'running databases')
ORDER BY rank DESC;

-- MySQL / InnoDB full-text search
CREATE FULLTEXT INDEX ft_body ON articles (title, body);
SELECT *, MATCH(title, body) AGAINST('running databases' IN NATURAL LANGUAGE MODE) AS score
FROM articles
WHERE MATCH(title, body) AGAINST('running databases' IN NATURAL LANGUAGE MODE)
ORDER BY score DESC;
```

**Example input / output:**

```text
Query intent: find documents about "running databases"

LIKE '%run%':
  matches title 'Running databases at scale' (has "Run")
  ALSO matches 'The concatenation trick' -> NO... "concatenation" has no "run"
  BUT '%run%' WOULD match "overrun", "running", "prune"? no -> "run" substring only.
  -> substring semantics, no ranking: rows returned in table order.

Full-text  @@ plainto_tsquery('english','running databases'):
  'running' -> stem 'run'; 'databases' -> stem 'databas'
  doc 1 contains run + databas  -> MATCH, ts_rank = 0.099 (both terms) -> ranked #1
  doc 2 contains neither stem   -> NO match
  -> word-aware + stemmed + ranked best-first.
```

**Data:**

```sql
CREATE TABLE articles (
    id    BIGSERIAL PRIMARY KEY,
    title TEXT,
    body  TEXT,
    sku   TEXT
);
-- Row examples:
-- (1, 'Running databases at scale', 'We ran the database ...', 'AB-100')
-- (2, 'The concatenation trick',    'Category theory notes ...', 'CD-200')
```

**Flow:**

1. Yes
2. No, natural language
3. No
4. Yes
5. Text search requirement
6. Exact substring / prefix / code?
7. LIKEprefix -> B-tree; anywhere ->pg_trgm index
8. Need fuzzy/typo, customrelevance,faceting, or huge scale?
9. DB full-text searchtsvector+GIN / FULLTEXT +ranking
10. Dedicated engineElasticsearch / OpenSearch

