---
id: "01_Databases_and_SQL/Query_Performance/Indexing/Min_Max_Statistics_and_Data_Skipping"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Query_Performance/Indexing"
tools: []
---

# Min Max Statistics and Data Skipping

**L4 Advanced** · [↑ Indexing](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Columnar files and lakehouse tables store the minimum and maximum of each column per file or row group, so a query engine can skip whole chunks that cannot match the predicate — a coarse, index-free form of indexing whose power depends entirely on how the data is sorted.

### 1. The big picture
A row-store database finds rows with B-tree indexes: fine-grained, per-row pointers, maintained on every write. Data lakes and warehouses hold immutable files (Parquet, ORC, Delta, Iceberg) scanned in bulk. Per-row indexes don't fit — files are rewritten, not updated in place, and the cost of maintaining an index per row would defeat bulk loading.

Instead the writer records **lightweight statistics** at each granularity: per file (in the table's transaction log/manifest), per **row group** (in the Parquet footer), and per **page**. The reader checks the statistics against the predicate first; if `max(age) < 50` and the query is `age > 50`, that chunk is **skipped without reading it**. This is *data skipping* / *predicate pushdown* / *zone maps* / *small materialized aggregates* — and it is the same idea as statistics-driven pruning in partitioned tables.

It only works if the min/max ranges are **narrow and non-overlapping**, which is a data-layout problem: sort or cluster the data by the columns you filter on.

### 2. Core concepts and vocabulary
- **Min/max statistics (zone map)** — smallest and largest value of a column in a chunk (plus null count, row count).
- **Data skipping** — avoiding reading chunks that statistics prove can't satisfy the predicate.
- **Row group / column chunk / page (Parquet)** — a file is split into row groups (e.g. 128 MB); each holds one column chunk per column; chunks contain pages. Statistics exist at chunk and page level.
- **Predicate pushdown** — evaluating filters at the storage reader using statistics/dictionary/bloom data, instead of after loading rows.
- **Partition pruning** — skipping directories/partitions whose partition key doesn't match; coarser and explicit.
- **Clustering / sort order / Z-order** — physically ordering data so related values land in the same files.
- **Bloom filter** — probabilistic set-membership structure in the file footer; answers "definitely not here" for equality on high-cardinality columns.
- **Compaction (`OPTIMIZE`)** — rewriting many small files into fewer well-sized, well-ordered files.
- **Table statistics (optimizer)** — row counts, distinct counts, histograms used for plan choice (different from file-skipping stats, though both are "statistics").

### 3. How it works, step by step
Writing:
1. The writer buffers rows into a row group, encodes each column.
2. For each column chunk (and page) it records min, max, null count.
3. Delta Lake additionally records per-file stats (record count, min/max/null count for the first 32 columns by default) in the transaction log JSON; Iceberg stores them in manifest files.

Reading `SELECT … WHERE age > 50`:
1. **Table level:** read the transaction log/manifest; discard files whose `maxValues.age <= 50` — no file open.
2. **File level:** open footer (small); discard row groups whose max ≤ 50.
3. **Page level:** skip pages by page index min/max.
4. Read only the surviving chunks, **only the projected columns** (columnar), decode, apply the predicate exactly to remaining rows.

Because statistics are *conservative*, skipping is safe: if a chunk might contain a match it is read. The savings come from chunks that provably cannot.

### 4. Worked example
```sql
-- Delta Lake (Spark SQL)
CREATE TABLE events (ts timestamp, user_id bigint, event_type string, payload string)
USING delta PARTITIONED BY (event_date date);

-- Unsorted ingest: every file has ts spanning the whole day -> overlapping ranges -> little skipping.
-- Compact and cluster so file ranges become narrow and (almost) disjoint:
OPTIMIZE events ZORDER BY (user_id, event_type);

-- Query: skip by partition (event_date) THEN by file min/max on user_id
SELECT count(*) FROM events
WHERE event_date = '2025-03-01' AND user_id = 42;

-- Inspect the effect (Spark UI / EXPLAIN FORMATTED / DESCRIBE DETAIL):
-- "number of files read" vs "number of files in table"; "data skipped".
```
File entry in the transaction log (abridged):
```json
{"add": {"path": "event_date=2025-03-01/part-0001.parquet",
         "stats": "{\"numRecords\": 4, \"minValues\": {\"user_id\": 10}, \"maxValues\": {\"user_id\": 30}}"}}
```
For `user_id = 42` this file is skipped without being opened. How to read the result: compare files/bytes scanned before and after `OPTIMIZE`; low skipping on a column means its ranges overlap across files (random layout) or statistics aren't collected for that column (beyond the first 32 by default).

### 5. Under the hood
- **Why sorting before writing matters.** If rows are sorted by `user_id` within a file set, each file covers a tight interval — e.g. [1–1000], [1001–2000] — and a point predicate matches at most one file. If randomly ordered, every file has min≈0 and max≈huge: nothing can be skipped even though statistics exist. Choosing the sort key: the column(s) most frequently used in selective filters, with enough cardinality to produce disjoint ranges; for *multiple* columns, lexicographic sorting only helps the leading column, while **Z-order / Hilbert curves** interleave bits so several columns get moderate locality at once (at the cost of weaker locality for each than a single-column sort).
- **Composite sort keys** behave like composite B-tree keys: filtering only on the second column gets little benefit because its values are spread across the whole range of the first.
- **File size trade-off.** Skipping granularity is the file/row-group. Smaller files → finer skipping but more metadata, more listing, more task overhead (the "small files problem"); typical target 128 MB–1 GB per file, row groups 64–512 MB (engine dependent). Metadata cost grows with file count; log checkpoints and manifest compaction keep planning fast.
- **Columns with unfriendly statistics.** Long strings are truncated in min/max; high-cardinality random IDs (UUIDv4) don't cluster; `LIKE '%x%'` and expressions on the column (UDFs, casts) block pushdown because the engine can't reason about min/max through opaque functions.
- **Bloom filters** (Parquet/ORC): for equality on high-cardinality columns where min/max is useless (UUIDs), a bloom filter in the footer answers "not here" with a tunable false-positive rate (more bits → fewer false positives → bigger footer).
- **ORC vs Parquet.** ORC keeps row-group indexes (every 10,000 rows by default) with min/max and optional bloom filters inside the file; Parquet has chunk stats and page indexes. Both support pushdown; ORC's built-in index is finer by default.
- **Relationship to B-tree indexes.** Zone maps are far smaller and cheaper to maintain but coarser: they narrow to a *set of chunks*, then you still scan inside them. B-trees locate individual rows. Migrating an indexed Postgres reporting table to a warehouse: the *index on the filter column* typically becomes a **sort/cluster key** (or partition key); indexes for uniqueness or point lookups just disappear (the warehouse doesn't enforce them) — compensate with data-quality checks.
- **Optimizer statistics.** Separately, the cost-based optimizer uses table/column statistics (row counts, distinct counts, histograms) for join order and strategy; in Spark, `ANALYZE TABLE … COMPUTE STATISTICS FOR COLUMNS` — a missing analyze leads to bad join order or broadcast decisions. Histograms capture skew (a few heavy values), which a single density number hides. PostgreSQL extended statistics handle correlated columns.

### 6. Performance and design implications
- **Measure skipping**, not just runtime: files/bytes scanned, rowgroups skipped, partitions pruned. Validate a new clustering/Z-order with before/after production-query metrics, not a synthetic benchmark.
- **Write path costs:** sorting/Z-ordering rewrites data (`OPTIMIZE` is compute-heavy); run incrementally on new partitions only; schedule off-peak.
- **Append-only, time-ordered data** gets skipping on the time column for free; late/out-of-order data widens ranges — periodic compaction restores them.
- **Don't over-partition:** high-cardinality partition keys create tiny files; partition by coarse date, cluster by the rest.
- **Keep stats columns in the first N** (Delta's 32) or configure `delta.dataSkippingNumIndexedCols` / `dataSkippingStatsColumns`.
- **Push predicates in sargable form:** compare raw columns to literals; avoid wrapping in UDFs.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `OPTIMIZE t [WHERE …] [ZORDER BY (cols)]` (Delta) | Compact + cluster files | Improve skipping on filter columns |
| `ALTER TABLE … CLUSTER BY (…)` / liquid clustering (Delta), `CLUSTER BY` (BigQuery, Snowflake clustering key) | Managed clustering | Evolving filter columns |
| `delta.dataSkippingNumIndexedCols` | How many leading columns get stats | Filter column beyond position 32 |
| Parquet `row_group_size`, `write_statistics`, bloom filter options | Writer layout | Pipeline design |
| `ANALYZE TABLE … COMPUTE STATISTICS FOR COLUMNS` (Spark) / `ANALYZE` (PG) | Optimizer stats | Join planning |
| `CREATE STATISTICS … (dependencies, ndistinct)` (PG) | Extended multi-column stats | Correlated predicates |
| Spark UI "number of files read", `EXPLAIN FORMATTED` PushedFilters | Verify pushdown/skipping | Always |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Large append-mostly tables filtered on few columns | Highly random point lookups needing per-row index (use a database/key-value store) |
| Sort by the most-filtered column | Many equally important filter columns with one lexicographic sort → Z-order/liquid clustering |
| Target file size 128 MB–1 GB | Streaming ingest producing thousands of tiny files without compaction |
| Bloom filters for high-cardinality equality | Range predicates (bloom can't help) |
| Partition by date, cluster within | Partitioning by user_id/high cardinality |

### 9. Common mistakes
- **Assuming statistics alone give skipping** → random layout → ranges overlap → sort/cluster/compact.
- **Z-ordering on many columns** → locality diluted → pick 2–4 most selective filter columns.
- **UDF/function in the predicate** → pushdown blocked → rewrite to raw column comparison.
- **Too many small files** → metadata and task overhead dominate → compaction and writer batching.
- **Confusing partition pruning with min/max skipping** → partitions are explicit directory values; min/max is per-file statistics.
- **Ignoring stats-column limits** → filter column past the indexed N has no stats → configure.
- **Forgetting `ANALYZE`** → optimizer picks bad joins → collect column statistics.

### 10. 🎯 Interview kit
**30-second answer:** Parquet and Delta/Iceberg store per-file and per-row-group min/max (and null counts). The engine compares them with the query's predicate and skips chunks that can't match — like a coarse index. It only pays off when the data is sorted or clustered on the filter columns so ranges are narrow and non-overlapping.

**Structure a longer answer:** why per-row indexes don't fit lakes → statistics hierarchy (table log → file → row group → page) → skipping algorithm → layout dependence (sort, Z-order, compaction, file size) → limits (functions, random IDs; bloom filters) → how to measure.

**Likely follow-ups:**
- *Why sort before writing Parquet?* → Narrow min/max ranges → more row groups/files skipped.
- *Target file size?* → ~128 MB–1 GB; balance skipping granularity vs metadata/small-files overhead.
- *How is Z-order like an index?* → Co-locates related values so stats prune across several columns; not per-row, needs rewriting.
- *Parquet vs ORC skipping?* → ORC has built-in row-group indexes and bloom filters by default; Parquet uses chunk/page stats and optional bloom filters.
- *UDF in WHERE?* → Opaque to pushdown; engine must read and evaluate.
- *Postgres indexes in the warehouse?* → Filter-column indexes become clustering/sort keys; uniqueness/point-lookup indexes vanish.

**Signals interviewers look for:** knowing skipping is layout-dependent, quantifying with files/bytes scanned, balancing file size and clustering cost.

### 11. Check yourself
- [ ] Explain skipping with a numeric example of five files and a predicate.
- [ ] Say why random ingestion order destroys skipping and how to fix it.
- [ ] Compare partition pruning, min/max skipping and bloom filters.
- [ ] Choose a sort key and file size for a 5 TB events table filtered by date and user.
- [ ] Describe how to prove a clustering change helped in production.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [Why does sorting data before writing Parquet dramatically improve later pruning, and how do you decide the sort key for a lake table?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20387_Why_does_sorting_data_before_writing_Parquet_dramatically_im.md>)
- [Intermediate] [How do you decide the target file size for a Parquet lake table so that skipping is effective but you avoid too many tiny files?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20411_How_do_you_decide_the_target_file_size_for_a_Parquet_lake_ta.md>)
- [Deep] [How does a Parquet file's column min/max statistics let a query engine skip row groups, and how is that like a coarse index?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20277_How_does_a_Parquet_file_s_column_min_max_statistics_let_a_qu.md>)
- [Deep] [How does a Delta Lake / Iceberg table's Z-ordering or data skipping resemble an index, and why is it useful for a data engineer?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20287_How_does_a_Delta_Lake_Iceberg_table_s_Z-ordering_or_data_ski.md>)
- [Deep] [What are table statistics and how does the optimizer use them?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19497_What_are_table_statistics_and_how_does_the_optimizer_use_the.md>)
- [Deep] [What is a histogram in table statistics and how does it help with skewed data?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19610_What_is_a_histogram_in_table_statistics_and_how_does_it_help.md>)
- [Deep] [Explain how Delta Lake data skipping uses min/max column statistics in the transaction log to prune Parquet files without any index.](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20373_Explain_how_Delta_Lake_data_skipping_uses_min_max_column_sta.md>)
- [Deep] [How does Parquet's internal structure (row groups, column chunks, page-level min/max stats) let engines skip data, and how is that different from a database index?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20386_How_does_Parquet_s_internal_structure_row_groups_column_chun.md>)

## Related concepts

- [Parquet Internals](<../../../02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Parquet_Internals.md>) *(File Formats)*
- [Large Values and TOAST](<../../SQL_Language/SQL_Fundamentals/Large_Values_and_TOAST.md>) *(SQL Fundamentals)*

## References

- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Maintaining Your Delta Lake › Z-Order Optimize
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Building Native Applications with Delta Lake › File statistics
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Performance Tuning Optimizing Your Data Pipelines with Delta Lake › Z-Ordering
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Performance Tuning Optimizing Your Data Pipelines with Delta Lake › Table Statistics
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 5 Statistics, Data Distribution, And Cardinality
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Storage Design Patterns › Solution
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Performance Tuning › Data Skipping
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Performance Tuning › ZORDER BY
- *Building Medallion Architectures* — Piethein Strengholt — Laying the Groundwork › V-Ordering
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Segment and rowgroup elimination

