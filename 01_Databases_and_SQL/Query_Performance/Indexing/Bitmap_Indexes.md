---
id: "01_Databases_and_SQL/Query_Performance/Indexing/Bitmap_Indexes"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Query_Performance/Indexing"
tools: ["Oracle", "PostgreSQL"]
---

# Bitmap Indexes

**L4 Advanced** · [↑ Indexing](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A bitmap index keeps one bit-vector per distinct value of a column and answers filters by AND/OR-ing those vectors, which is superb for low-cardinality columns in read-mostly data warehouses and terrible for write-heavy OLTP.

### 1. The big picture
B-tree indexes work best when values are **high cardinality** — many distinct values, each matching few rows (`customer_id`, `email`). They work poorly for something like `gender`, `status` (5 values), or `is_active`: each value matches a large share of the table, so the index doesn't reduce work and costs space.

Bitmap indexes flip the idea. For each *distinct value* they store a bitmap with one bit per row: bit *i* is 1 if row *i* has that value. A query such as

```sql
WHERE region = 'EMEA' AND tier = 'GOLD' AND churned = 'N'
```

becomes a bitwise AND of three bitmaps, an operation the CPU does 64 rows at a time. Combining many weak filters is exactly what data-warehouse queries do, which is why bitmap indexes became a signature feature of Oracle warehouses and why equivalent ideas reappear in columnstore engines and in PostgreSQL's executor.

### 2. Core concepts and vocabulary
- **Cardinality** — number of distinct values in a column; *low* cardinality means few distinct values.
- **Bitmap** — an array of bits, one per row (or row range), marking membership.
- **Bitwise operations** — `AND`, `OR`, `NOT`, `XOR` on bitmaps implement `AND`, `OR`, `NOT`, `<>` predicates.
- **Bitmap index (persistent)** — bitmaps stored on disk as the index (Oracle, some warehouse engines).
- **Bitmap scan (transient)** — PostgreSQL builds an in-memory bitmap of matching row locations at query time from ordinary indexes.
- **Bitmap join index** — Oracle index on a fact table keyed by a dimension attribute.
- **Run-length / word-aligned compression (RLE, WAH, Roaring)** — encodings that shrink sparse or clustered bitmaps while allowing fast bitwise operations.
- **Bitmap filter (SQL Server)** — a runtime bit-array built from the build side of a hash join to discard probe-side rows early; not a stored index.
- **Delete bitmap (columnstore)** — a structure marking logically deleted rows in compressed row groups.

### 3. How it works, step by step
**Persistent bitmap index (Oracle-style)**
1. Table has rows numbered by physical order. Column `status` has values `{NEW, PAID, SHIPPED, CANCELLED}`.
2. The index stores one compressed bitmap per value, plus a small B-tree on the value to find it.
3. `WHERE status = 'PAID'` loads one bitmap, converts set bits to row IDs, fetches the rows.
4. `WHERE status IN ('NEW','PAID') AND region = 'EU'` ORs two bitmaps, ANDs with region's bitmap, then fetches rows.
5. `COUNT(*)` can be answered from the bitmap alone, with no table access.

**PostgreSQL bitmap scan**
1. A normal B-tree index scan returns matching TIDs; with multiple indexes (`BitmapAnd`/`BitmapOr`) each produces a bitmap of heap pages.
2. The bitmaps are combined in memory.
3. A `Bitmap Heap Scan` reads the pages in physical order (sequential-ish I/O) and rechecks conditions when the bitmap became *lossy* (page-level rather than tuple-level) due to `work_mem`.

### 4. Worked example
```sql
-- Oracle: warehouse fact table, 200 M rows
CREATE BITMAP INDEX bx_sales_region ON sales(region);     -- 8 distinct values
CREATE BITMAP INDEX bx_sales_channel ON sales(channel);   -- 4 distinct values
CREATE BITMAP INDEX bx_sales_status ON sales(status);     -- 5 distinct values

SELECT COUNT(*)
FROM   sales
WHERE  region = 'EMEA' AND channel = 'ONLINE' AND status IN ('PAID','SHIPPED');
-- Plan: BITMAP AND of three BITMAP OR/ACCESS operations -> SORT AGGREGATE
--       No TABLE ACCESS step: the count comes from the bitmaps.
```

Size intuition: with 8 values and 200 M rows, an *uncompressed* region bitmap set is 8 × 200 M bits ≈ 200 MB; with RLE on clustered data it's far less. A B-tree on the same column would hold 200 M entries (several GB).

The OLTP trap:

```sql
-- Session 1                           -- Session 2
UPDATE sales SET status='PAID'         UPDATE sales SET status='SHIPPED'
 WHERE order_id = 1001;                 WHERE order_id = 1002;   -- BLOCKS (!)
```

Both rows' bits live in the same compressed bitmap segment; modifying one locks a whole range of rows' bits, so unrelated updates serialize.

Disambiguation: SQL Server has no persistent bitmap index. You'll see "bitmap" in plans as `Bitmap Create` / `Bitmap` filters on hash joins (star-join optimisation) and as the *delete bitmap* of columnstore indexes.

### 5. Under the hood
- **Structure.** A bitmap index entry is `(key value, start rowid, end rowid, compressed bitmap)`. The segments let the engine skip empty regions. Because bitmaps cover contiguous row-ID ranges, physical clustering of values makes RLE very effective.
- **Why DML is expensive.** Changing a row changes two bitmaps (clears one bit, sets another); locking is at the *bitmap segment* granularity, which covers many rows. Concurrent single-row DML therefore blocks, and each change may require re-compression. Warehouses avoid this by loading in batches and rebuilding or marking indexes unusable during load.
- **Operator support.** Bitmaps handle equality, `IN`, `AND`, `OR`, `NOT` and `IS NULL` (B-trees typically can't index NULL entries; bitmaps can). They don't give sort order, so range predicates and `ORDER BY` still prefer B-trees.
- **Star transformation.** The optimizer rewrites a star-join into subqueries on dimension tables, uses bitmap indexes on fact foreign keys, ANDs the bitmaps, and fetches only qualifying fact rows.
- **Columnstore relationship.** Columnstore stores each column in compressed segments with dictionary + RLE + bit-packing (a value needing 7 bits is stored in 7, not 32), and per-segment min/max for elimination; batch-mode execution then processes vectors with SIMD, achieving what bitmap indexes achieve (fast scans of low-cardinality columns) without the DML locking problem. Deletes are soft: a **delete bitmap** per partition flags rows, which is fast but leaves space until maintenance.
- **Roaring bitmaps** (used by Lucene, Druid, Pinot, ClickHouse) split the row space into chunks and pick array/bitmap/run containers per chunk — a modern alternative with fast operations.
- **Not to be confused** with *bitmap version vectors* in distributed anti-entropy: a different use of bitmaps to track which writes a replica has seen.

### 6. Performance and design implications
- Good for: data warehouses, read-mostly dimensions, many ad-hoc filter combinations on low-cardinality columns, `COUNT(*)`.
- Poor for: OLTP, high-concurrency updates, high-cardinality columns (bitmaps become sparse and large without compression benefits), range predicates.
- Rule of thumb: distinct values below ~1 % of rows (often far lower); selectivity of the combined filter must still be meaningful, otherwise a scan wins.
- Load patterns: drop/disable, bulk-load, rebuild — or use partition exchange.
- Check usage with the execution plan (`BITMAP CONVERSION`, `BITMAP AND`) and size via segment views; track DML lock waits.
- On PostgreSQL don't look for a bitmap index type; make sure ordinary indexes on the filter columns exist and `work_mem` is large enough to keep bitmaps exact.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE BITMAP INDEX … ON t(col)` (Oracle) | Persistent bitmap index | Warehouse low-cardinality columns |
| `STAR_TRANSFORMATION_ENABLED = TRUE` (Oracle) | Enable star transformation | Star schemas |
| `ALTER INDEX … UNUSABLE / REBUILD` | Disable during loads | Batch ETL |
| `EXPLAIN` → `BitmapAnd`, `Bitmap Heap Scan` (PostgreSQL) | Inspect transient bitmaps | Combining indexes |
| `work_mem` | Keeps bitmap exact (not lossy) | Large bitmap scans |
| `CREATE CLUSTERED COLUMNSTORE INDEX` (SQL Server) | Compressed column storage | Analytics on SQL Server |
| `ALTER INDEX … REORGANIZE` on columnstore | Compress/clean delta and deletes | Maintenance |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Few distinct values, many filter combinations, read-mostly | Rows are inserted/updated one at a time by many sessions |
| Counts and existence checks | You need ordered results or range predicates |
| Fact-table foreign keys in a star schema | Columns are high cardinality (e.g. IDs) |
| Columnstore on engines without bitmap indexes | Frequent small updates to the columnstore (delta store churn) |

### 9. Common mistakes
- Creating bitmap indexes on an OLTP table → lock waits, "mysterious" blocking → use B-trees, or columnstore for analytic copies.
- Indexing a high-cardinality column → huge index with no benefit → check distinct counts.
- Expecting SQL Server or PostgreSQL to offer persistent bitmap indexes → plan on columnstore, partitioning, or ordinary indexes.
- Row-by-row ETL against bitmap-indexed tables → slow loads → bulk load and rebuild.
- Forgetting that a combined filter that is not selective will still scan → check estimated rows before indexing.
- Confusing deleted-rows bitmaps in columnstore with indexes → space not reclaimed → run maintenance.

### 10. 🎯 Interview kit
**30-second answer:** A bitmap index stores one bit-vector per distinct value; predicates become bitwise AND/OR on those vectors. It's fast and compact for low-cardinality columns in read-mostly warehouses, but changes lock whole bitmap segments, so it's wrong for OLTP.

**Structure a longer answer:** the cardinality problem with B-trees → bitmap representation and operations → compression → DML cost → engine specifics (Oracle persistent; PostgreSQL transient bitmap scans; SQL Server columnstore/bitmap filters).

**Likely follow-ups:**
- *Why not use bitmap indexes in OLTP?* → concurrent DML serialises on bitmap segments.
- *B-tree vs bitmap for `status` with 5 values?* → bitmap for warehouse combinations; B-tree or partial index for OLTP point filters.
- *How does PostgreSQL use bitmaps?* → `BitmapAnd/Or` of index results then heap scan in physical order.
- *What is a delete bitmap?* → soft-delete marker in columnstore.

**Signals interviewers look for:** you tie index type to cardinality and workload, and you know the engine-specific equivalents.

### 11. Check yourself
- [ ] Describe how three equality filters become one bitmap and a count.
- [ ] Explain why updating one row can block another.
- [ ] State when a B-tree beats a bitmap on a low-cardinality column.
- [ ] Explain RLE compression's effect on clustered values.
- [ ] Say what SQL Server offers instead of persistent bitmap indexes.
<!-- explainer:end -->

## Implementations (tools)

- [Oracle](<../../../TOOLS/Oracle/_index.md>)
- [PostgreSQL](<../../../TOOLS/PostgreSQL/_index.md>)

## Practice questions

- [Intermediate] [Explain why a bitmap index on a gender column is efficient but a bitmap index on an email column would be a disaster.](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-00524_Explain_why_a_bitmap_index_on_a_gender_column_is_efficient_b.md>)
- [Deep] [When is a bitmap index the right choice over a B-tree index and why does column cardinality drive that decision?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-00515_When_is_a_bitmap_index_the_right_choice_over_a_B-tree_index_.md>)
- [Deep] [Explain how bitmap indexes compress and store row references, and why that makes them memory-efficient for low-cardinality columns.](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-00528_Explain_how_bitmap_indexes_compress_and_store_row_references.md>)
- [Deep] [How would you decide between a filtered/partial index and a bitmap index for a column that's 95% one value and 5% another?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-00548_How_would_you_decide_between_a_filtered_partial_index_and_a_.md>)
- [Deep] [Explain why a bitmap index's storage can grow non-linearly as cardinality creeps up, and how you'd monitor for that.](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-00580_Explain_why_a_bitmap_index_s_storage_can_grow_non-linearly_a.md>)
- [Deep] [A team proposes bitmap indexes to speed up dashboard filters on a table taking 5k writes/sec — how do you evaluate the concurrency risk and steer them to the right structure without blocking their launch?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-00629_A_team_proposes_bitmap_indexes_to_speed_up_dashboard_filters.md>)
- [Deep] [How do you decide when to push filtering into a bitmap/columnar layer at the storage engine versus keeping composite B-tree indexes at the table for a hybrid HTAP workload?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-00665_How_do_you_decide_when_to_push_filtering_into_a_bitmap_colum.md>)
- [Expert] [Explain the difference between a dense and sparse bitmap and how the optimizer estimates cost across them.](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-00556_Explain_the_difference_between_a_dense_and_sparse_bitmap_and.md>)

## Related concepts

- [Leftmost Prefix Rule](<../../SQL_Language/SQL_Fundamentals/Leftmost_Prefix_Rule.md>) *(SQL Fundamentals)*
- [Index Access Paths](<../../Database_Internals/DBMS_Concepts/Index_Access_Paths.md>) *(DBMS Concepts)*
- [Index Tuning in Practice](<Index_Tuning_in_Practice/_index.md>) *(Indexing)*

## References

- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Data Storage, Indexing, and Replication › Bitmap indexes
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Index arChIteCture R HTE TURE
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Compression › Columnstore Compression Algorithms
- *Database Internals* — Alex Petrov — Anti-Entropy and Dissemination › Bitmap Version Vectors
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — The Views

