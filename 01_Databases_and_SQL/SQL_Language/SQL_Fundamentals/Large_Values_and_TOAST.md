---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Large_Values_and_TOAST"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: []
---

# Large Values and TOAST

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Fixed-size pages can't hold huge values, so engines move large text/JSON/blob data out of line (TOAST in PostgreSQL, off-row/LOB pages in SQL Server) — which makes wide columns cheap to skip but expensive to fetch, so never `SELECT` them casually, and choose compact types for keys.

### 1. The big picture
Databases store rows in fixed-size pages (8 KB in PostgreSQL and SQL Server). A row must generally fit in a page, yet users store multi‑KB JSON documents, long text, and binary blobs. The engine's answer: keep a small pointer in the main row and put the big value elsewhere. That design explains several everyday surprises:
- A `SELECT *` on a table with a big JSON column is dramatically slower than selecting only needed columns.
- Updating one small column can still be cheap even if the row has a large document — or not, depending on engine.
- A key stored as `varchar(36)` rather than a 16‑byte `uuid`/`binary(16)` bloats every index and slows joins.
- Deep `OFFSET` pagination reads and discards enormous numbers of rows (and their large values).

### 2. Core concepts and vocabulary
- **Page / block** — the unit of I/O and caching (8 KB by default in PG and SQL Server).
- **TOAST** (*The Oversized-Attribute Storage Technique*) — PostgreSQL's mechanism to compress and/or move values over ~2 KB into a side table.
- **Inline vs out-of-line** — value stored in the main row vs in a separate structure referenced by pointer.
- **Row-overflow / LOB data** — SQL Server's equivalents: `varchar(max)`, `nvarchar(max)`, `varbinary(max)`, `xml` stored in LOB pages; variable columns may spill to *row-overflow* pages.
- **Wide row** — a row whose in-row size is large, reducing rows per page and cache efficiency.
- **Covering / included columns** — indexes that satisfy a query without touching the base row.
- **JSONB** — PostgreSQL binary JSON, indexable (GIN) and TOAST-able.
- **Keyset (seek) pagination** — paging using a "last seen key" predicate instead of `OFFSET`.

### 3. How it works, step by step
**PostgreSQL TOAST**
1. When a row's tuple exceeds the TOAST threshold (~2 KB), PG tries to **compress** the largest TOAST-able columns (pglz or lz4).
2. If still too big, it **slices** values into ~2 KB chunks stored in the table's hidden `pg_toast.pg_toast_<oid>` table, and leaves a small pointer in the main tuple.
3. Reads that don't mention the column never touch the TOAST table. Reads that do must fetch and reassemble (and decompress) chunks — extra I/O and CPU.
4. Updates that don't change the TOASTed column keep the same pointer (no rewrite of the large value) — one reason a wide document column can coexist with frequent small updates.

**SQL Server**
1. `varchar(max)`/`nvarchar(max)` values up to 8,000 bytes stay in-row by default; larger values go to **LOB pages** with a pointer (table option `large value types out of row` controls this).
2. Fixed/variable columns that overflow 8,060 bytes push variable-length columns to **row-overflow** pages.
3. Reading pointers costs random page reads on top of the base page read.

### 4. Worked example
```sql
-- PostgreSQL: see how big the table is vs how much is TOAST
SELECT relname,
       pg_size_pretty(pg_relation_size(c.oid))                AS heap,
       pg_size_pretty(pg_relation_size(c.reltoastrelid))      AS toast,
       pg_size_pretty(pg_total_relation_size(c.oid))          AS total
FROM   pg_class c WHERE relname = 'documents';

-- Slow: drags the TOASTed payload back for every row, then ignores it
SELECT * FROM documents WHERE tenant_id = 7 ORDER BY created_at DESC LIMIT 50;

-- Fast: list view needs only light columns; fetch payload on demand
SELECT id, title, created_at FROM documents
WHERE tenant_id = 7 ORDER BY created_at DESC LIMIT 50;

-- Keyset pagination instead of OFFSET 100000
SELECT id, title, created_at FROM documents
WHERE tenant_id = 7 AND (created_at, id) < (:last_created, :last_id)
ORDER BY created_at DESC, id DESC LIMIT 50;
```
Reading the result: `EXPLAIN (ANALYZE, BUFFERS)` shows far fewer buffer hits for the narrow select, because the TOAST relation isn't read. For `OFFSET 100000`, the plan must produce and discard 100,000 rows before returning 50, so latency grows with page number; keyset's cost is constant with a composite index `(tenant_id, created_at, id)`.

SQL Server equivalent: `SELECT *` on a table with `nvarchar(max)` shows extra logical reads attributed to *LOB logical reads* in `SET STATISTICS IO`.

**UUID as text:** `uuid` in PG is 16 bytes; the same value as `varchar(36)` is ~37 bytes plus collation-aware comparison. Every secondary index entry carries the key, so indexes bloat ~2–3× and fewer entries fit per page → deeper trees, more cache misses, slower joins.

### 5. Under the hood
- **Tuple layout.** A PG heap tuple has a ~23-byte header; TOAST pointers are ~18 bytes. Strategy per column: `PLAIN` (no TOAST), `MAIN` (compress, avoid out-of-line), `EXTERNAL` (out-of-line, no compression — good for substring ops on text), `EXTENDED` (default: compress then out-of-line). Change with `ALTER TABLE … ALTER COLUMN … SET STORAGE`.
- **Compression** lowers I/O but burns CPU; already-compressed data (images, zipped blobs) gains nothing and should be `EXTERNAL`.
- **JSON querying.** Extracting a field from a large JSONB forces detoast + parse of the whole document. Index specific paths with expression indexes or generated columns (`CREATE INDEX ON t ((payload->>'status'))`) or a GIN index for containment; in SQL Server use computed columns on `JSON_VALUE(...)` and index those.
- **Columnstore** (SQL Server): values are dictionary-compressed per column segment; large strings can hit dictionary size limits and degrade compression.
- **Migrating a column type in place** on a big table: add a new column, backfill in batches, add triggers or dual-write to keep in sync, add index concurrently, swap names/reads, drop the old column. A direct `ALTER COLUMN TYPE` rewrites the table under an exclusive lock.
- **Lake analogy:** small-file problems in Iceberg/Delta come from many small writes; compaction rewrites files to a target size without blocking readers (snapshot isolation).

### 6. Performance and design implications
- Select only what you need; list endpoints should never fetch payloads.
- Split rarely-read big values into a **side table** (1:1) keyed by the main id — keeps the hot table dense and cache-friendly. Or store blobs in object storage and keep a URL/hash.
- Keep keys compact (`int`, `bigint`, `uuid`/`binary(16)`); prefer time-ordered UUIDs (UUIDv7/ULID) to avoid random insertion page splits.
- Bound value sizes with constraints; unbounded text/JSON invites outliers that dominate I/O.
- Measure: `pg_total_relation_size`, `pg_stat_statements` (`shared_blks_read`), `sys.dm_db_partition_stats` (`lob_used_page_count`, `row_overflow_used_page_count`), `STATISTICS IO`.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ALTER COLUMN … SET STORAGE EXTERNAL/MAIN` | Control TOAST strategy | Tune substring/compress behavior |
| `pg_relation_size(reltoastrelid)` | Size of out-of-line data | Diagnose bloat |
| `default_toast_compression = lz4` | Faster compression (PG14+) | CPU-bound detoast |
| `sp_tableoption 'large value types out of row'` | Force LOBs off-row | Wide rows with max types |
| Expression / generated-column index | Index JSON field | Filter on JSON attribute |
| Keyset predicate | Constant-time paging | Infinite scroll, APIs |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Large JSON in the row | Queries filter on JSON fields often without indexes |
| Side table for payloads | You always read payload with the parent (extra join) |
| Object storage + reference | Transactional consistency with the blob matters |
| Native `uuid`/`binary` keys | Legacy systems demand text IDs |
| Keyset pagination | Users need random page jumps |

### 9. Common mistakes
- `SELECT *` over wide tables → needless detoasting and network → list columns explicitly.
- UUID as `varchar(36)`/`char(36)` → bloated indexes and slow joins → native type or binary(16).
- Deep `OFFSET` paging → O(offset) latency → keyset pagination.
- Filtering JSON without an index → full detoast scans → expression/GIN index.
- Treating TOAST as free → hidden I/O → monitor TOAST relation size and access.

### 10. 🎯 Interview kit
**30-second answer:** Pages are small and fixed, so big values are stored out of line — TOAST in Postgres, LOB/overflow pages in SQL Server. Selecting them costs extra I/O and decompression, so you select only needed columns, keep keys compact, and avoid deep OFFSET.
**Structure a longer answer:** page-size constraint → out-of-line mechanism → read/write cost → design fixes (narrow selects, side tables, indexes on extracted fields, keyset paging).
**Likely follow-ups:**
- *Why is `SELECT *` slow with a big text column?* → detoasting and larger result transfer even if the app ignores it.
- *Why is varchar UUID bad?* → 2×+ bytes in every index, collation compares, poorer cache density.
- *How to migrate type in place safely?* → shadow column + batched backfill + swap.
- *Why does infinite scroll get slower?* → OFFSET reads and discards; use keyset.
**Signals interviewers look for:** page-level reasoning, concrete measuring tools, practical redesigns.

### 11. Check yourself
- [ ] Explain TOAST in four steps.
- [ ] Show how to prove a table's size is mostly TOAST.
- [ ] Rewrite an OFFSET paging query as keyset with its index.
- [ ] Estimate the index size difference between `uuid` and `varchar(36)` for 100M rows.
- [ ] Outline a zero-downtime column type migration.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [Why does storing a UUID as varchar bloat indexes and slow joins compared to a native uuid/binary type, and how would you migrate the column type in place on a large table?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39938_Why_does_storing_a_UUID_as_varchar_bloat_indexes_and_slow_jo.md>)
- [Intermediate] [Your frontend infinite-scroll feed calls a search endpoint that gets slower as users scroll. Explain, from a full-stack view, how deep OFFSET kills the DB and how you'd redesign the API contract for keyset paging.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40310_Your_frontend_infinite-scroll_feed_calls_a_search_endpoint_t.md>)
- [Deep] [Your queries filter jsonb_col ->> 'status' constantly and it's slow. Compare a GIN index, an expression B-tree index on the extracted key, and promoting status to a real column for equality versus containment.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39945_Your_queries_filter_jsonb_col_-_status_constantly_and_it_s_s.md>)
- [Deep] [What is clustering or a sort key in a warehouse table, and how does it help range-filter queries on a large fact table?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40023_What_is_clustering_or_a_sort_key_in_a_warehouse_table_and_ho.md>)
- [Deep] [Your Redshift query spilled to disk because of an under-sized VARCHAR distribution. Explain DISTKEY and SORTKEY choice, and how a wrong DISTSTYLE causes broadcast or redistribution during a JOIN.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40113_Your_Redshift_query_spilled_to_disk_because_of_an_under-size.md>)
- [Deep] [A table has a JSONB column and you filter on jsonb_column ->> 'status' constantly. Compare a GIN index, an expression B-tree index on the extracted key, and promoting status to a real column. Which for high-selectivity equality vs. containment queries?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40463_A_table_has_a_JSONB_column_and_you_filter_on_jsonb_column_-_.md>)
- [Deep] [How would you write SQL to compute a per-entity feature vector as a JSON/array column for cheap serving fetch, and what are the trade-offs versus one column per feature?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40682_How_would_you_write_SQL_to_compute_a_per-entity_feature_vect.md>)
- [Expert] [Why is the entity-attribute-value (EAV) pattern often an anti-pattern, and what are its query costs?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40240_Why_is_the_entity-attribute-value_EAV_pattern_often_an_anti-.md>)

## Related concepts

- [MySQL Optimizer Behavior](<../../../TOOLS/MySQL/MySQL_Platform/MySQL_Optimizer_Behavior.md>) *(MySQL Platform)*
- [Advanced Aggregation](<../Aggregation/Advanced_Aggregation.md>) *(Aggregation)*
- [Index Access Paths](<../../Database_Internals/DBMS_Concepts/Index_Access_Paths.md>) *(DBMS Concepts)*

## References

- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Infrastructure Operations › Points to remember
- *Data Engineering Design Patterns* — Bartosz Konieczny — A Wrapper in a Table? › Size
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Security Design Patterns › Problem
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Storage Design Patterns › Problem
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Storage Design Patterns › Data Representation
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — What are Columnstore Indexes RE COLUMNSTORE INDEXES › Rowgroups and Segments Rowgroups and Segments
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Compression › Columnstore Compression Algorithms
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Compression › Row Order (Vertipaq) Optimization
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore metadata › Columnstore Index Memory Usage
- *Designing Data-Intensive Applications* — Martin Kleppmann — Storage and Retrieval › Column-oriented storage and column families
- *Designing Data-Intensive Applications* — Martin Kleppmann — Graphs and Iterative Processing › The move toward declarative query languages
- *Designing Data-Intensive Applications* — Martin Kleppmann — Q › R
- *Designing Data-Intensive Applications* — Martin Kleppmann — S
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › Storing JSON Data
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › Performance and Indexing with JSON
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › PolyBase
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Performance Considerations with PolyBase
- *The Enterprise Big Data Lake* — Alex Gorelik — Cataloging the Data Lake › Profiling hierarchical data
- *The Enterprise Big Data Lake* — Alex Gorelik — Big Data in Medicine › C
- *The Enterprise Big Data Lake* — Alex Gorelik — Big Data in Medicine › J

