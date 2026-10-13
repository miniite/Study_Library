---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/RAG_Metrics_in_SQL"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: []
---

# RAG Metrics in SQL

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A RAG (retrieval-augmented generation) system is, operationally, a set of relational tables — documents, chunks, embeddings, queries, retrievals — and ordinary SQL (normalization, anti-joins, window functions, hashing, parameterized queries) gives you its integrity checks and quality metrics.

### 1. The big picture
RAG pipelines ingest documents, split them into **chunks**, embed each chunk as a vector, and at question time retrieve the most similar chunks to feed an LLM. The vector index is only one part: around it sits plain relational data that must stay correct and observable — which documents were ingested, which chunks have embeddings, what each query retrieved and how similar it was.

Applying standard SQL fundamentals here pays off quickly: normalize so a title isn't repeated per chunk, find orphans with anti-joins, detect duplicate chunks with hashes, measure retrieval confidence with window functions, and filter safely with parameterized queries. This lesson is SQL fundamentals *applied* to that domain.

### 2. Core concepts and vocabulary
- **Document** — a source unit (PDF, page, ticket) with metadata.
- **Chunk** — a contiguous slice of a document (by tokens/sentences) that is embedded and retrieved.
- **Embedding** — fixed-length vector of floats representing a chunk's meaning.
- **Retrieval log** — record of each query, the chunks returned, rank and similarity score.
- **Top-k / similarity score** — nearest-neighbour results and their cosine similarity (or distance).
- **Orphan** — child row whose parent no longer exists (or parent with no children).
- **Content hash** — digest of normalized text used for exact-duplicate detection.
- **Token count** — chunk length in tokens; embedding models truncate at a max.
- **Parameterized query** — SQL with bound parameters rather than concatenated strings.
- **Metadata filter** — predicate (tenant, language, date) applied alongside vector search.

### 3. How it works, step by step
**A minimal normalized schema**
```sql
documents(document_id PK, title, source_uri, tenant_id, ingested_at)
chunks(chunk_id PK, document_id FK→documents, chunk_no, text, token_count, text_hash)
embeddings(chunk_id PK/FK→chunks, model, vector, embedded_at)
queries(query_id PK, asked_at, text)
retrievals(query_id FK, chunk_id FK, rank, score, PRIMARY KEY(query_id, rank))
```
1. **Normalize:** title lives once in `documents`; `chunks` carries only `document_id`. This avoids update anomalies (renaming a document touches one row).
2. **Ingest atomically:** insert the document and all its chunks in **one transaction** so a failure doesn't leave a half-ingested document.
3. **Guard integrity:** FK constraints plus unique `(document_id, chunk_no)`.
4. **Observe:** run recurring SQL checks — orphans, missing embeddings, oversize chunks, duplicates, low-confidence retrievals.

### 4. Worked example
```sql
-- 1) Documents with zero embedded chunks (LEFT JOIN + IS NULL)
SELECT d.document_id, d.title
FROM   documents d
LEFT   JOIN chunks c      ON c.document_id = d.document_id
LEFT   JOIN embeddings e  ON e.chunk_id    = c.chunk_id
GROUP  BY d.document_id, d.title
HAVING COUNT(e.chunk_id) = 0;

-- 2) Orphan chunks (no parent document) — anti-join
SELECT c.chunk_id
FROM   chunks c
WHERE  NOT EXISTS (SELECT 1 FROM documents d WHERE d.document_id = c.document_id);

-- 3) Chunks silently truncated at embedding (limit 8192 tokens)
SELECT chunk_id, token_count
FROM   chunks WHERE token_count > 8192;

-- 4) Exact-duplicate chunks: keep the earliest, flag the rest
SELECT chunk_id
FROM (
  SELECT chunk_id,
         ROW_NUMBER() OVER (PARTITION BY text_hash ORDER BY chunk_id) AS rn
  FROM   chunks) x
WHERE rn > 1;

-- 5) Top-1 similarity per query -> spot low-confidence retrievals
SELECT AVG(score) AS avg_top1,
       SUM(CASE WHEN score < 0.55 THEN 1 ELSE 0 END) * 1.0 / COUNT(*) AS low_conf_rate
FROM   retrievals
WHERE  rank = 1;

-- 6) Safe metadata filter (Python/psycopg style)
--    cur.execute("SELECT chunk_id FROM chunks c JOIN documents d USING(document_id)
--                 WHERE d.tenant_id = %s AND d.ingested_at >= %s", (tenant, since))
```
Reading it: query 1 uses `COUNT(e.chunk_id)` — which ignores NULLs — so documents whose chunks all lack embeddings return 0. Query 4 deduplicates *before* embedding: duplicates waste vector storage and crowd the top-k with identical text, lowering diversity. Query 5's `rank = 1` filter is the retriever's best hit; a falling average is an early warning of corpus drift or query–corpus mismatch.

### 5. Under the hood
- **Hashing for dedup:** normalize (lowercase, collapse whitespace, strip boilerplate) before hashing, else trivially different text hashes differently. Exact hashes catch identical chunks; near-duplicates need MinHash/SimHash or embedding similarity.
- **Vector storage:** pgvector stores `vector(n)` columns (16-bit or 32-bit floats; TOASTed when large) with `ivfflat`/`hnsw` indexes — approximate nearest neighbour, trading recall for speed; filters applied after an ANN scan can return fewer than k rows ("post-filter problem"), so tenant filters often need partitioning or an iterative/filtered-ANN setting.
- **Anti-join semantics:** `NOT IN (subquery)` fails if the subquery yields NULL (see NULL Handling); `NOT EXISTS` or `LEFT JOIN … IS NULL` are safe and planned as anti-joins.
- **Window functions** execute after `WHERE`/`GROUP BY`; to filter on `ROW_NUMBER()` you must wrap in a subquery/CTE or use `QUALIFY` where supported.
- **Parameterization** keeps user text (query strings, filter values) out of SQL syntax: the statement is parsed once and values are bound as data, which also enables plan reuse. Identifiers (column names) can't be bound; whitelist them.
- **Referential integrity at scale:** large vector tables often skip FKs for ingest speed; replace with periodic orphan audits.

### 6. Performance and design implications
- Index `chunks(document_id)`, `chunks(text_hash)`, and `retrievals(query_id)`; keep `embeddings` separate so wide vectors don't bloat the hot chunk table (see Large Values and TOAST).
- Keep chunk text and vector in sync via `embedded_at`/`model` columns, so model upgrades can find stale rows: `WHERE model <> 'current'`.
- Store retrieval logs append-only and partition by time for cheap retention.
- Sample for quality: join `retrievals` to human-labelled relevance to compute hit-rate@k, MRR — all expressible with window functions and conditional aggregation.
- Never build filter strings by concatenation; always bind parameters.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `LEFT JOIN … IS NULL` / `NOT EXISTS` | Find missing/orphan rows | Integrity audits |
| `ROW_NUMBER() OVER (PARTITION BY hash …)` | Dedup | Pre-embedding cleanup |
| `COUNT(col)` vs `COUNT(*)` | Count present values | Coverage metrics |
| `CREATE INDEX … USING hnsw` (pgvector) | ANN index | Fast similarity |
| Bound parameters (`%s`, `@p`, `$1`) | Injection-safe filters | Always |
| Unique/FK constraints | Integrity | Parent–child tables |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Normalized documents/chunks | Read path needs title per chunk (join cost; or denormalize deliberately) |
| Exact hash dedup | Near-duplicates dominate (need fuzzy) |
| FK constraints | Bulk ingest throughput is critical |
| Store vectors in Postgres | Billions of vectors / very high QPS (dedicated vector DB) |
| Log every retrieval | Privacy/PII in query text (mask or expire) |

### 9. Common mistakes
- Repeating the full document title on each chunk → update anomalies, wasted space → normalize.
- Ingesting document and chunks in separate transactions → partial documents → one transaction.
- Concatenating user text into SQL filters → injection → bind parameters.
- Silent truncation of long chunks → missing content in embeddings → monitor `token_count`.
- Embedding duplicates → crowded top-k → dedup by normalized hash first.
- `COUNT(*)` over a LEFT JOIN → counts the NULL row as 1 → `COUNT(child.id)`.

### 10. 🎯 Interview kit
**30-second answer:** Treat the RAG corpus as relational data: normalized documents → chunks → embeddings, loaded transactionally, audited with anti-joins and hashes, with retrieval logs analysed through window functions, and all filters parameterized.
**Structure a longer answer:** schema → integrity (constraints, atomic ingest) → data-quality SQL (orphans, missing embeddings, oversize, duplicates) → retrieval metrics → safety (parameters) → scaling (indexes, partitions, vector index).
**Likely follow-ups:**
- *How find documents never embedded?* → LEFT JOIN chain, `HAVING COUNT(e.chunk_id)=0`.
- *Why dedupe before embedding?* → storage, and identical hits reduce answer diversity.
- *Parameterized vs sanitized strings?* → binding separates code from data; sanitizing is fragile.
- *Why would a filtered ANN search return too few rows?* → post-filtering after the approximate scan.
**Signals interviewers look for:** reusing fundamentals in a new domain, correctness checks, security awareness.

### 11. Check yourself
- [ ] Sketch the normalized RAG schema and justify each key.
- [ ] Write the orphan, missing-embedding, and oversize-chunk queries.
- [ ] Explain why duplicates hurt retrieval quality.
- [ ] Compute low-confidence rate from a retrieval log.
- [ ] Show a parameterized metadata filter and explain why it's injection-safe.
<!-- explainer:end -->

## Practice questions

- [Basic] [How do you store a document's text chunks in a SQL table alongside a reference to their source document for a RAG pipeline?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40726_How_do_you_store_a_document_s_text_chunks_in_a_SQL_table_alo.md>)
- [Basic] [What is normalization, and how would you normalize a table that repeats the full document title on every chunk row?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40798_What_is_normalization_and_how_would_you_normalize_a_table_th.md>)
- [Intermediate] [How would you write a query to find orphan fact rows whose dimension key does not exist in the dimension table?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39994_How_would_you_write_a_query_to_find_orphan_fact_rows_whose_d.md>)
- [Intermediate] [How would you use a LEFT JOIN and IS NULL to find documents that have zero embedded chunks in your knowledge base?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40745_How_would_you_use_a_LEFT_JOIN_and_IS_NULL_to_find_documents_.md>)
- [Intermediate] [Write a query to compute the average similarity score of the top-retrieved chunk per query to spot low-confidence retrievals.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40770_Write_a_query_to_compute_the_average_similarity_score_of_the.md>)
- [Intermediate] [How would you use a parameterized query when building the metadata filter for a vector search to stay safe from injection?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40823_How_would_you_use_a_parameterized_query_when_building_the_me.md>)
- [Intermediate] [How would you query a document-chunk table to detect chunks whose token count far exceeds the embedding model's max input, which would have been silently truncated at ingest?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40914_How_would_you_query_a_document-chunk_table_to_detect_chunks_.md>)
- [Deep] [How would you deduplicate a RAG corpus in SQL before embedding, removing near-identical chunks that share a normalized-text hash, and why does embedding duplicates waste retrieval quality?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40845_How_would_you_deduplicate_a_RAG_corpus_in_SQL_before_embeddi.md>)

## Related concepts

- [Inverted Indexes](<../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Inverted_Indexes.md>) *(Data Structures and Algorithms)*
- [Chunk Size and Retrieval Quality](<../../../11_AI_and_ML/LLM_Applications/Retrieval_and_RAG/Chunk_Size_and_Retrieval_Quality.md>) *(Retrieval and RAG)*
- [Re-ranking](<../../../11_AI_and_ML/LLM_Applications/Retrieval_and_RAG/Re-ranking.md>) *(Retrieval and RAG)*
- [MySQL Optimizer Behavior](<../../../TOOLS/MySQL/MySQL_Platform/MySQL_Optimizer_Behavior.md>) *(MySQL Platform)*
- [UDF Performance](<../SQL_Functions/UDF_Performance.md>) *(SQL Functions)*
- [Classic SQL Query Problems](<../../Database_Internals/DBMS_Concepts/Classic_SQL_Query_Problems.md>) *(DBMS Concepts)*

## References

- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Getting the Structured Back into SQL › Chapter 39. Getting the "Structured" Back into SQL
- *Fuzzy Data Matching with SQL* — Jim Lehmer — Names, Names, Names › Full Name
- *Fuzzy Data Matching with SQL* — Jim Lehmer — Code-Generating Code › Gather Together Every Code "Artifact" You Can
- *Fuzzy Data Matching with SQL* — Jim Lehmer — Import Artifacts into SQL
- *Fuzzy Data Matching with SQL* — Jim Lehmer — Import Artifacts into SQL › Load all SQL artifacts (views, stored procs, user-defined functions) into the \
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Technical interview questions › Summary
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Data types › 3.4.3 Avoiding the overhead by storing data as XML
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Indexes › 11.3 #69 Reorganizing indexes to fix page density
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Building on Subqueries, Common Table Expressions, and Unions › Using EXCEPT and INTERSECT
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Thinking About Performance › Database Cleanup
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Working with XML and JSON › The QUERY Method
- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Managing Spark Clusters › Understanding the SQL section

