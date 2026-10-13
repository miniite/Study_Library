---
type: "practice"
difficulty: "Intermediate"
---

# What is full-text search and when would you use something like Elasticsearch?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Pagination Strategies](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Pagination_Strategies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: Two things bundled together — (1) what is full-text search (FTS) as a concept, and (2) when to reach for a dedicated engine like Elasticsearch instead of your primary database. The interviewer is probing whether you understand the difference between "matching a value in a column" (a database WHERE / LIKE) and "ranking documents by relevance against natural-language queries," and whether you know the operational cost of introducing a second data store.

What the interviewer is looking for (intermediate level): a precise definition of FTS (inverted index, tokenization/analysis, relevance scoring), a clear list of when it's the right tool, and — the part that separates a strong candidate — the trade-offs: consistency, cost, and the fact that Elasticsearch is a secondary index you sync into, not your system of record.

Clarifying questions I'd ask: What's the scale (documents, QPS)? Do we need typo tolerance, synonyms, faceting, autocomplete? Is the corpus text-heavy (product descriptions, logs, docs) or mostly exact-match lookups? Is near-real-time indexing acceptable, or do we need read-your-own-writes?

Plan: define FTS from first principles → contrast with a plain DB query → explain how Elasticsearch works internally → enumerate concrete "when to use / when not to use" scenarios → cover the operational trade-offs.

What full-text search is

Full-text search is the ability to search within the natural-language content of documents and return results ranked by relevance, not just rows that exactly match a value. If a user types red running shoes, FTS should find products described as "crimson trainers for jogging" and rank the best matches first — something a SQL WHERE description = 'red running shoes' can never do, and even LIKE '%red%' does poorly (no ranking, no stemming, full table scan).

The core data structure is the inverted index. Instead of mapping document → words, it maps word → list of documents (and positions) that contain it. Building it requires an analysis / tokenization pipeline:

Tokenization — split text into terms ("Red Running Shoes!" → red, running, shoes).
Normalization — lowercase, strip punctuation, remove accents.
Stop-word handling — optionally drop common words like "the", "a".
Stemming / lemmatization — reduce words to a root so running, ran, runs all match run.
Synonyms — map sneaker → shoe.

At query time the same analysis runs on the query, then the engine intersects/unions the posting lists and scores each candidate. The classic scoring model is TF-IDF, and modern Elasticsearch/Lucene default to BM25: a term matters more if it's frequent in this document (TF) but rare across the whole corpus (IDF), with a saturation curve so a word appearing 100 times isn't 100× more relevant than appearing once.

On top of the index, FTS engines add fuzzy matching (typo tolerance via edit distance), phrase / proximity search (using stored term positions), faceting/aggregations (counts per category — "42 Nike, 18 Adidas"), highlighting, and autocomplete.

How Elasticsearch fits

Elasticsearch is a distributed engine built on Apache Lucene (which provides the inverted index and BM25). Elasticsearch adds:

Sharding — an index is split into shards spread across nodes so it scales horizontally.
Replication — each shard has replicas for availability and read throughput.
A JSON/REST API and rich query DSL (bool queries, filters, aggregations).
Near-real-time indexing (documents become searchable ~1s after write, on refresh).

Critically, Elasticsearch is almost always a secondary store. Your source of truth stays in Postgres/MySQL/DynamoDB; you replicate the searchable fields into Elasticsearch (via change-data-capture, a queue, or dual writes). That's the mental model to communicate: system of record vs. search index.

When you WOULD use Elasticsearch (or FTS in general)
Keyword search over large text corpora with relevance ranking: product catalog search, job listings, documentation, support tickets.
Typo tolerance, synonyms, stemming are required (e-commerce search, autocomplete).
Faceted search / filters at scale — "shoes, under ₹3000, brand=Nike, 4★+" with live counts.
Log and event analytics (the ELK stack): ingest millions of log lines, search and aggregate them (error rate by service in last 15 min). This is arguably ES's biggest real-world use.
Geo + text combined queries: "coffee shops near me matching 'espresso'".
High read QPS on search where you don't want to hammer the primary DB with expensive LIKE/ILIKE scans.
When you would NOT reach for it
Exact-match key lookups ("get order by ID") — that's your primary DB's job; ES adds nothing but sync lag.
You need strong consistency / read-your-writes — ES is near-real-time and eventually consistent with your DB.
Small scale / simple needs — Postgres has built-in FTS (tsvector/tsquery, GIN indexes) that handles millions of rows with ranking and no extra infrastructure. Reaching for a whole Elasticsearch cluster for 50k rows is over-engineering. Databases like SQLite (FTS5) and MySQL also ship FTS.
Transactions / joins are the core need — ES is document-oriented, not relational; multi-document ACID transactions aren't its model.
The trade-offs a strong answer emphasizes
It's a second system to keep in sync. You now own a replication pipeline (CDC/Debezium, Kafka, or dual writes), plus reindexing when the mapping changes. Sync bugs = stale/missing search results.
Eventual consistency. The DB commit and the ES index update aren't atomic. Design for it (e.g., re-fetch canonical data from the DB after search returns IDs).
Operational cost. ES is memory/heap-hungry, needs careful shard sizing, and clusters can go yellow/red. Managed offerings (Elastic Cloud, AWS OpenSearch) reduce but don't remove this.
Not a system of record. Don't store data only in ES; a reindex or cluster loss must be recoverable from the primary store.
Alternatives exist. Postgres FTS for moderate scale; Typesense / Meilisearch for simpler developer-friendly search; Algolia as a hosted SaaS if you'd rather not run infra; OpenSearch as the open-source ES fork.
Summary / what a strong answer emphasizes

Full-text search = an inverted index + an analysis pipeline (tokenize, stem, synonyms) + a relevance scoring model (BM25), giving ranked, typo-tolerant, faceted search over natural-language content — something plain SQL equality/LIKE can't do. Use Elasticsearch when you have large text corpora needing relevance ranking, typo tolerance, faceting, or log analytics at high QPS. Treat it as a secondary, eventually-consistent index synced from your source-of-truth DB. Don't use it for exact-key lookups, when you need strong consistency, or when Postgres FTS already suffices — and always name the operational/consistency cost of running a second store.

#### Common mistakes
calling ES a database (system of record); forgetting to mention the sync pipeline and eventual consistency; recommending ES for tiny datasets where Postgres FTS wins; not knowing what an inverted index or BM25 is.

#### Follow-up
probes to expect: "How would you keep ES in sync with Postgres?" (CDC/Debezium → Kafka → indexer), "What's an inverted index?", "How does relevance scoring work?", and "How would you build autocomplete?" (edge n-grams / completion suggester).

**Comparison:**

```
Capability	SQL = / LIKE '%..%'	Postgres FTS (tsvector)	Elasticsearch
Relevance ranking	No	Yes (ts_rank)	Yes (BM25, tunable)
Stemming / synonyms	No	Yes	Yes (rich analyzers)
Typo tolerance (fuzzy)	No	Limited	Yes (edit distance)
Faceting / aggregations	Manual GROUP BY	Manual	Yes (native aggs)
Horizontal scale	DB-bound	DB-bound	Sharded/replicated
Log analytics at scale	No	No	Yes (ELK)
Extra infra to run	None	None	A cluster to operate
System of record	Yes	Yes	No (secondary index)
Consistency	Strong	Strong	Near-real-time / eventual
Situation	Recommended tool
Get row by primary key / exact match	Primary DB
Moderate-scale ranked search, few million rows	Postgres FTS
Large catalog, typo tolerance, facets, high QPS	Elasticsearch / OpenSearch
Log & metrics search + dashboards	Elasticsearch (ELK)
Don't want to run infra	Algolia / Typesense Cloud
```

**Code:**

```

// Elasticsearch: a bool query — must-match text (scored) + filter (not scored), // with an aggregation (facet) for brand counts. GET /products/_search { "query": { "bool": { "must": [ { "match": { "description": "red running shoes" } } ], "filter": [ { "range": { "price": { "lte": 3000 } } }, { "term": { "brand": "nike" } } ] } }, "aggs": { "by_brand": { "terms": { "field": "brand" } } } }

CODE
Copy

```sql
-- Postgres built-in full-text search (the lighter alternative to Elasticsearch)
-- One-time: create a GIN index over the searchable text.
CREATE INDEX idx_products_fts
  ON products USING GIN (to_tsvector('english', description));

-- Ranked search with stemming ('running' matches 'run'):
SELECT id, name,
       ts_rank(to_tsvector('english', description),
               plainto_tsquery('english', 'red running shoes')) AS rank
FROM products
WHERE to_tsvector('english', description) @@ plainto_tsquery('english', 'red running shoes')
ORDER BY rank DESC
LIMIT 20;
```

**Example input / output:**

```text
Query:  "red runing shose"     (note the typos)
Analysis (query time): lowercase -> tokenize -> stem -> ["red", "run", "shoe"] (fuzzy edit-distance ≤2)

Corpus:
  doc1: "Crimson running shoes for jogging"   -> terms after analysis: red? no; run, shoe  (via synonym crimson→red, stem running→run)
  doc4: "Blue running trainers"                -> run, shoe(trainer synonym)
  doc7: "Red formal leather shoes"             -> red, shoe

Scoring (BM25, higher = more relevant, "shoe" rare-ish → high IDF):
  doc1: matches red(syn)+run+shoe  -> score 8.9   -> rank 1
  doc7: matches red+shoe           -> score 5.2   -> rank 2
  doc4: matches run+shoe           -> score 4.7   -> rank 3

Result (ranked): [doc1, doc7, doc4]
=> Typos tolerated, stemming + synonyms applied, results ordered by relevance —
   none of which a SQL `WHERE description = 'red running shoes'` would return.
```

**Data:**

```json
// Inverted index (conceptual): term -> postings (docId : positions)
{
  "red":     [ { "doc": 1, "pos": [0] }, { "doc": 7, "pos": [3] } ],
  "run":     [ { "doc": 1, "pos": [1] }, { "doc": 4, "pos": [0] } ],  // 'running' stemmed to 'run'
  "shoe":    [ { "doc": 1, "pos": [2] }, { "doc": 4, "pos": [1] }, { "doc": 7, "pos": [4] } ]
}
// Query "red running shoes" -> analyze to [red, run, shoe]
// -> intersect postings -> doc 1 matches all 3 -> highest BM25 score -> ranked first.
```

**Flow:**

1. CDC / Debezium / queue
2. analyze + index docs
3. match + filter + BM25
4. ranked doc IDs + highlights
5. optional re-fetch canonicalrows
6. Write to Primary DBsystem of record
7. Indexer service
8. (Elasticsearchinverted index, sharded)
9. User query'red running shoes'
10. Search API
11. Ranked results + facets to user

