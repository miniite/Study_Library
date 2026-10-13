---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Vector_Similarity_in_SQL"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: []
---

# Vector Similarity in SQL

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Vector similarity search stores embeddings (arrays of floats that encode meaning) in a column and finds the nearest ones to a query vector with an `ORDER BY distance LIMIT k`, usually through an approximate nearest-neighbour index (HNSW or IVFFlat) that trades a little recall for large speed.

### 1. The big picture
An **embedding** model turns text, images or other items into a fixed-length vector (e.g. 768 or 1536 floats) so that semantically similar items are close together in vector space. Retrieval-augmented generation (RAG), semantic search, recommendations and deduplication all reduce to: *given a query vector, return the k closest stored vectors*, often with a metadata filter (tenant, document type, date).

Relational databases can do this natively now: PostgreSQL via the `pgvector` extension, SQL Server 2025 with a built-in `VECTOR` type and `VECTOR_DISTANCE`, plus vector support in MySQL HeatWave, Oracle 23ai, Snowflake, BigQuery and others. Keeping vectors next to the relational data gives you transactional consistency, joins, row-level security and a single backup, at the cost of the specialized scaling of dedicated vector engines.

Related topics: full-text (keyword) search with `tsvector` and GIN, geospatial nearest-neighbour, ranking with window functions, and index-assisted `ORDER BY … LIMIT`.

### 2. Core concepts and vocabulary
- **Embedding** — a dense float vector produced by a model; the dimension is fixed per model.
- **Distance metric** — how closeness is measured: **L2 (Euclidean)**, **cosine distance** (1 − cosine similarity), **inner product**.
- **Nearest neighbour (kNN)** — exact search: compare the query with every row.
- **Approximate nearest neighbour (ANN)** — index-based search that may miss some true neighbours.
- **Recall@k** — fraction of the true top-k that the ANN result contains.
- **HNSW** — Hierarchical Navigable Small World graph index.
- **IVFFlat** — inverted-file index: cluster vectors into lists, search the closest lists.
- **Normalization** — scaling vectors to length 1; then cosine, inner product and L2 give the same ranking.
- **Hybrid search** — combining keyword relevance and vector similarity.
- **Reciprocal rank fusion (RRF)** — merging ranked lists by summing `1/(k + rank)`.
- **MRR** — mean reciprocal rank, a retrieval-quality metric.

### 3. How it works, step by step
**Storing and querying (pgvector).**
1. `CREATE EXTENSION vector;` and declare a column `embedding vector(1536)`.
2. Insert vectors produced by the embedding model (the model is called by your application, not by SQL).
3. Query with a distance operator and `ORDER BY` + `LIMIT`:
   - `<->` L2 distance, `<=>` cosine distance, `<#>` negative inner product.
4. Without an index this is an exact scan (O(N·d)); correct but slow at scale.
5. Create an ANN index matching the operator class (`vector_cosine_ops`, `vector_l2_ops`, `vector_ip_ops`). The planner uses it only when the `ORDER BY` expression uses the same operator.

**Which metric?** Use the metric the embedding model was trained for (most text embedding models are cosine / normalized). For unit-length vectors, `<#>` (inner product) is the cheapest and gives the same order as cosine.

**HNSW vs IVFFlat.**
- *HNSW* builds a multi-layer proximity graph; a search descends layers greedily and keeps a candidate list of size `ef_search`. High recall and speed, slower build, more memory, can be built on an empty table.
- *IVFFlat* runs k-means to create `lists` clusters; a query probes `probes` nearest lists. Faster to build, smaller, but needs representative data before building and has lower recall unless `probes` is raised.

### 4. Worked example
```sql
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE chunks (
  chunk_id    bigserial PRIMARY KEY,
  tenant_id   int         NOT NULL,
  doc_type    text        NOT NULL,
  doc_version int         NOT NULL,
  body        text        NOT NULL,
  embedding   vector(1536) NOT NULL,
  tsv         tsvector GENERATED ALWAYS AS (to_tsvector('english', body)) STORED
);

CREATE INDEX chunks_hnsw ON chunks
  USING hnsw (embedding vector_cosine_ops) WITH (m = 16, ef_construction = 64);
CREATE INDEX chunks_tsv  ON chunks USING gin (tsv);
CREATE INDEX chunks_tenant ON chunks (tenant_id, doc_type);

-- 1) Top-5 nearest chunks to a query vector ($1)
SET hnsw.ef_search = 100;                        -- more candidates → better recall, slower
SELECT chunk_id, body, embedding <=> $1 AS cosine_distance
FROM   chunks
ORDER  BY embedding <=> $1
LIMIT  5;

-- 2) Tenant-scoped retrieval
SELECT chunk_id, body
FROM   chunks
WHERE  tenant_id = 42 AND doc_type = 'policy'
ORDER  BY embedding <=> $1
LIMIT  5;
```

Why the filter matters: with an HNSW/IVFFlat index the filter is applied *after* the index returns its candidates (post-filtering). If tenant 42 owns 1% of rows and `ef_search = 40`, the ANN stage may return 40 candidates of which none or one belongs to tenant 42, so you receive fewer than 5 rows or poor results. Fixes: raise `ef_search`; use pgvector's iterative scan (0.8+) so the index keeps scanning until enough filtered rows are found; partition the table or build partial indexes per large tenant; or, for highly selective filters, let the planner use the B-tree on `tenant_id` and do an exact scan of the few matching rows.

Hybrid search with reciprocal rank fusion:

```sql
WITH vec AS (
  SELECT chunk_id, ROW_NUMBER() OVER (ORDER BY embedding <=> $1) AS r
  FROM chunks ORDER BY embedding <=> $1 LIMIT 50
),
kw AS (
  SELECT chunk_id,
         ROW_NUMBER() OVER (ORDER BY ts_rank_cd(tsv, websearch_to_tsquery('english', $2)) DESC) AS r
  FROM chunks
  WHERE tsv @@ websearch_to_tsquery('english', $2)
  ORDER BY ts_rank_cd(tsv, websearch_to_tsquery('english', $2)) DESC LIMIT 50
)
SELECT COALESCE(v.chunk_id, k.chunk_id) AS chunk_id,
       COALESCE(1.0/(60 + v.r), 0) + COALESCE(1.0/(60 + k.r), 0) AS rrf_score
FROM   vec v FULL OUTER JOIN kw k ON k.chunk_id = v.chunk_id
ORDER  BY rrf_score DESC, chunk_id          -- chunk_id breaks ties deterministically
LIMIT  10;
```

Measure ANN recall against an exact scan on a sample:

```sql
-- exact neighbours (disable index use for this transaction)
BEGIN; SET LOCAL enable_indexscan = off;
CREATE TEMP TABLE exact AS
SELECT q.id AS qid, c.chunk_id, ROW_NUMBER() OVER (PARTITION BY q.id ORDER BY c.embedding <=> q.embedding) r
FROM sample_queries q CROSS JOIN LATERAL (SELECT * FROM chunks ORDER BY embedding <=> q.embedding LIMIT 10) c;
COMMIT;
-- then compare with the same query run with the index; recall@10 = matches / (10 * #queries)
```

Retrieval-quality metric stored per request: `MRR = AVG(1.0 / rank_of_gold_chunk)` over logged queries.

### 5. Under the hood
- **Cost of exact search.** N vectors × d dimensions of multiply-adds per query; 1 M × 1536 floats ≈ 6 GB scanned. Storage per vector ≈ 4·d bytes (pgvector `vector`; `halfvec` uses 2 bytes, `bit` for binary quantization).
- **HNSW internals.** Each vector gets a random max layer; upper layers are sparse "highways". Search enters at the top, greedily moves to the closest neighbour, descends, and at layer 0 explores `ef_search` candidates. `m` controls edges per node (memory and recall), `ef_construction` controls build quality. The graph must usually fit in memory/shared buffers to be fast; updates insert nodes, deletes mark them and need vacuuming/reindex to reclaim.
- **IVFFlat internals.** Build samples vectors, runs k-means to get `lists` centroids (rule of thumb: rows/1000 up to 1 M rows, sqrt(rows) beyond), then assigns each vector to its closest centroid. A query probes the closest `probes` lists (rule of thumb sqrt(lists)).
- **Planner interplay.** The index returns candidates in approximate order; filters apply afterwards; `LIMIT` is required for the index to be considered. Mixing operator and operator class (e.g. `<->` query on a cosine index) falls back to a sequential scan.
- **Silent degradation.** Mixing normalized and non-normalized vectors, or changing the embedding model without re-embedding, breaks distances without raising errors; recall can also drop gradually as data drifts away from IVFFlat's trained centroids. Monitor with the exact-vs-ANN check and norm statistics (`SELECT AVG(vector_norm(embedding))`).
- **SQL Server 2025** exposes `VECTOR_DISTANCE('cosine', a, b)` and a `VECTOR(n)` type, with approximate indexing introduced as a preview feature; design guidance (metric match, post-filter caveats, recall checks) is the same.

### 6. Performance and design implications
- Tune recall/latency with `ef_search` (HNSW) or `probes` (IVFFlat); measure recall on real queries, not guesses.
- Build indexes after bulk loading for IVFFlat; for HNSW raise `maintenance_work_mem` to build in memory.
- Keep embeddings in a separate table if the base row is wide and updated often; vectors are large and rewrite cost is high.
- Track `embedded_at` vs `updated_at` to find stale embeddings (`WHERE updated_at > embedded_at`) and re-embed incrementally.
- Invalidate caches (semantic caches, retrieval caches) when the source chunk version changes, using the version column as part of the key.
- For leaderboard ranks use `RANK() OVER (…)`; for geospatial nearest neighbours use a bounding-box prefilter and a GiST/PostGIS index before exact distance.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `vector(n)`, `halfvec(n)` | Vector column types | Store embeddings |
| `<->`, `<=>`, `<#>` | L2 / cosine / inner-product distance | Ranking |
| `USING hnsw (col vector_cosine_ops)` | HNSW index | Default ANN choice |
| `USING ivfflat … WITH (lists = N)` | IVFFlat index | Faster build, smaller index |
| `hnsw.ef_search`, `ivfflat.probes` | Query-time recall knobs | Tune recall vs latency |
| `hnsw.iterative_scan` (0.8+) | Keep scanning for filtered queries | Selective filters |
| `to_tsvector`, `@@`, `ts_rank_cd` + GIN | Full-text search | Keyword half of hybrid |
| `VECTOR_DISTANCE()` | Distance in SQL Server 2025 | SQL Server vector queries |
| `EXPLAIN (ANALYZE, BUFFERS)` | Confirm index use | Debug slow retrieval |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Vectors in the existing relational DB — one system, joins, ACID, row security | Hundreds of millions of vectors or very high QPS — dedicated vector store may scale better |
| HNSW — best recall/latency, ok to spend memory and build time | Memory-constrained, frequent heavy updates |
| IVFFlat — quick build, smaller | Data distribution shifts; low-recall risk without tuning |
| Exact scan — small tables or highly selective filters | Millions of rows and tight latency |
| Hybrid (keyword + vector) — names, IDs, rare terms matter | Pure semantic queries where keyword adds noise |
| Postgres full-text search — good until relevance tuning, faceting, language analyzers outgrow it | Need advanced analyzers, distributed search → Elasticsearch/OpenSearch |

### 9. Common mistakes
- Query operator doesn't match the index operator class → sequential scan → align `<=>` with `vector_cosine_ops`.
- Naive `WHERE tenant_id = …` with ANN → too few or wrong rows, or cross-tenant leakage if the filter is forgotten → scoped query + iterative scan/partitioning, and enforce row-level security.
- Building IVFFlat on an empty or tiny table → poor centroids → build after loading representative data.
- Forgetting `LIMIT` → planner can't use the index → always limit.
- Storing a mix of normalized and raw vectors under a cosine/IP setup → silently wrong ranking → normalize at write time and assert norms.
- Changing the embedding model without re-embedding → distances meaningless → version embeddings and re-embed all.
- Judging quality by eyeballing → regressions unnoticed → log gold chunks, compute recall@k and MRR.

### 10. 🎯 Interview kit
**30-second answer:** Store embeddings in a `vector(n)` column and retrieve with `ORDER BY embedding <=> $query LIMIT k`; add an HNSW or IVFFlat index for approximate search, tuned with `ef_search`/`probes`, and measure recall against an exact scan. Metadata filters are applied after the index, so selective filters need iterative scan, bigger candidate lists, partitioning or an exact plan.

**Structure a longer answer:** what embeddings are → metrics and normalization → exact vs ANN and index types → filtering and hybrid search → operations (re-embedding, recall monitoring) → when to leave the database.

**Likely follow-ups:**
- *Cosine vs L2?* → same ranking for normalized vectors; use the model's metric.
- *Why do I get fewer than k rows with a filter?* → post-filtering on ANN candidates.
- *HNSW vs IVFFlat?* → recall/latency/memory/build trade-off.
- *How do you combine BM25-style and vector scores?* → RRF or weighted normalized scores.
- *How do you detect recall regression?* → compare to brute-force on a sampled query set.

**Signals interviewers look for:** index/operator alignment, post-filter awareness, measurement of recall, operational plans for model/version changes, honest limits of in-database search.

### 11. Check yourself
- [ ] Write the nearest-5-chunks query and the index that supports it.
- [ ] Explain why a tenant filter can return too few rows with HNSW and list three fixes.
- [ ] State when cosine, L2 and inner product give the same ranking.
- [ ] Describe `lists`/`probes` and `m`/`ef_search` and what each trades.
- [ ] Write an exact-vs-ANN recall@10 check and an MRR query.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [How does pgvector let you run a nearest-neighbour similarity search in SQL, and what does the ORDER BY clause look like?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40749_How_does_pgvector_let_you_run_a_nearest-neighbour_similarity.md>)
- [Intermediate] [How would you combine a metadata WHERE filter with a vector similarity ORDER BY in a single pgvector query, and why is the filter useful for RAG?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40751_How_would_you_combine_a_metadata_WHERE_filter_with_a_vector_.md>)
- [Deep] [What is the difference between cosine distance and L2 distance operators in pgvector, and which fits normalized embeddings?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40752_What_is_the_difference_between_cosine_distance_and_L2_distan.md>)
- [Deep] [How would you store hybrid-search data so you can combine a SQL full-text (keyword) score with a vector similarity score for reranking?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40827_How_would_you_store_hybrid-search_data_so_you_can_combine_a_.md>)
- [Deep] [How would you design a query and index to efficiently answer 'the 10 nearest stores to this lat/long'? Compare naive haversine, bounding-box prefilter, and PostGIS/geospatial indexes.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40368_How_would_you_design_a_query_and_index_to_efficiently_answer.md>)
- [Deep] [How would you compute geospatial features like nearest-store distance in SQL for a delivery-time model, and when do you prefer a bounding-box prefilter over full haversine over all rows?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40680_How_would_you_compute_geospatial_features_like_nearest-store.md>)
- [Deep] [How would you combine a pgvector similarity search with a metadata WHERE filter (tenant_id, doc_type) so retrieval stays tenant-scoped, and why can a naive filter defeat the vector index?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40833_How_would_you_combine_a_pgvector_similarity_search_with_a_me.md>)
- [Deep] [How would you store per-request retrieval sets and compute Mean Reciprocal Rank of the gold chunk across queries in SQL for a retrieval-quality dashboard?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40923_How_would_you_store_per-request_retrieval_sets_and_compute_M.md>)

## Related concepts

- [Inverted Indexes](<../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Inverted_Indexes.md>) *(Data Structures and Algorithms)*
- [Full-text Search in Postgres](<../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Full-text_Search_in_Postgres.md>) *(PostgreSQL Platform)*
- [Similarity Search Complexity](<../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Similarity_Search_Complexity.md>) *(Data Structures and Algorithms)*

## References

- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — Q
- *The Enterprise Big Data Lake* — Alex Gorelik — Big Data in Medicine › U
- *Building Data Integration Solutions* — Jay Borthen — Key Concepts in Data Integration › Vector databases
- *Building Data Integration Solutions* — Jay Borthen — Data Stores and Management Systems › LanceDB
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Using Actual vs. Estimated Metrics for Operators
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Comparing Plans
- *Fuzzy Data Matching with SQL* — Jim Lehmer — CustomerCountByState View › S
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — P

