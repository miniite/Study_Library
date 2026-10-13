---
id: "01_Databases_and_SQL/Query_Performance/Indexing/Partitioning_and_Data_Skipping_in_Lakes"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Query_Performance/Indexing"
tools: ["Apache_Spark", "Hadoop", "Databricks"]
---

# Partitioning and Data Skipping in Lakes

**L4 Advanced** · [↑ Indexing](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Lakes and columnar warehouses have no B-tree indexes; they make queries fast by *not reading* data — partition pruning skips whole folders, and min/max file or row-group statistics skip individual files — so physical layout (partitions, file size, sort/cluster order) is the index.

### 1. The big picture
In a row store such as SQL Server, an index lets the engine *find* rows without scanning the table. In a data lake (Parquet/ORC files on S3, ADLS or HDFS, managed by Hive, Spark, Athena, Trino, Delta, Iceberg) or a warehouse (Snowflake, BigQuery, Redshift) there is usually no per-row lookup structure. Tables are large, immutable column files; the cost driver is **bytes scanned**. The optimizer's job becomes: "which files can I prove contain no matching rows, and skip?"

That proof comes from two layers of metadata:
1. **Partitioning** — data physically grouped by the values of one or a few columns (usually a date), each group in its own directory or file set. A filter on the partition column lets the planner list only matching directories (*partition pruning*).
2. **Data skipping** — each file (or each row group / micro-partition) carries min/max (and sometimes null counts, bloom filters) per column. A filter whose range doesn't overlap a file's min/max lets the engine skip that file without opening it.

Partitioning is coarse, explicit and designed by you; data skipping is fine-grained and automatic but only works if data is *clustered* so that min/max ranges are narrow. Both break down when you create too many tiny files.

### 2. Core concepts and vocabulary
- **Partition** — a subset of a table stored together, keyed by column value(s) (e.g. `dt=2026-10-01/`).
- **Hive-style partitioning** — directory names encode values: `s3://lake/sales/year=2026/month=10/`.
- **Partition pruning** — the planner skips partitions whose key cannot satisfy the predicate. *Static* when the predicate has literals; *dynamic* when the values come from a join at runtime.
- **Dynamic partition pruning (DPP)** — filter on a small dimension table is turned into a runtime filter on the large fact table's partition column (star-schema joins).
- **Data skipping** — using per-file min/max/null-count statistics to avoid reading files.
- **Row group / stripe** — unit inside a Parquet/ORC file with its own statistics (≈ 128 MB default in Parquet writers).
- **Micro-partition** — Snowflake's automatic 50–500 MB (uncompressed) immutable unit with metadata per column.
- **Clustering / Z-order / sort key** — physically ordering data so related values are co-located and min/max ranges are tight.
- **Small-file problem** — thousands of files of a few KB/MB: heavy listing and open overhead and weak statistics.
- **Compaction (`OPTIMIZE`, `rewrite_data_files`)** — rewriting many small files into fewer large ones.
- **Bloom filter index** — probabilistic per-file structure answering "this value is definitely not here" for equality predicates.
- **Bucketing** — hash-distributing rows into a fixed number of files so joins/aggregations on that key avoid a shuffle.
- **Sargable predicate** — a filter written so the engine can use the metadata (column compared to a constant, no function wrapped around it).

### 3. How it works, step by step
**Partition pruning**
1. You write `WHERE event_date BETWEEN '2026-09-01' AND '2026-09-07'`.
2. The planner asks the catalog/metastore for the partitions of the table and keeps only those whose `event_date` value satisfies the predicate.
3. Only files in those 7 directories are listed, opened and scanned. The other 3 years never get touched, not even their metadata.

**File-level data skipping**
1. Within the surviving partitions, the engine reads each file's footer (Parquet) or the table's transaction log/manifest (Delta, Iceberg), which hold min/max per column.
2. For predicate `customer_id = 4711`, any file with `min > 4711` or `max < 4711` is eliminated.
3. Remaining files are read column-wise (*projection pushdown* — only requested columns) and row groups inside them are again checked by their statistics (*predicate pushdown*).

**Dynamic partition pruning** (fact joined to filtered dimension)
1. Run the dimension side first: `dim_date WHERE year = 2026 AND quarter = 3` → a small set of `date_key` values.
2. Broadcast that set as a runtime filter into the fact-table scan.
3. The scan prunes fact partitions/files whose `date_key` isn't in the set, so the join processes far fewer rows.

How the systems differ:
| System | "Index-like" mechanism | You control it with |
|---|---|---|
| Hive/Spark/Athena | Directory partitions + Parquet stats | `PARTITIONED BY`, file size, `ORDER BY`/`sortWithinPartitions` |
| Delta Lake | Transaction log stats per file, Z-ORDER, bloom filters | `OPTIMIZE … ZORDER BY`, partition columns, `VACUUM` |
| Iceberg | Manifests with partition + column stats, *hidden partitioning* (transforms like `days(ts)`) | partition spec, `rewrite_data_files`, sort order |
| Snowflake | Automatic micro-partitions, optional clustering key | `CLUSTER BY`, `SYSTEM$CLUSTERING_INFORMATION` |
| BigQuery | Partitioned + clustered tables | `PARTITION BY`, `CLUSTER BY`, `require_partition_filter` |
| Redshift | Sort keys + zone maps (per-block min/max) | `SORTKEY`, `VACUUM` |

### 4. Worked example
Spark SQL over a Delta table of 3 TB of events:

```sql
-- Layout: partition by day, cluster within by customer_id
CREATE TABLE events (
  event_ts     TIMESTAMP,
  event_date   DATE GENERATED ALWAYS AS (CAST(event_ts AS DATE)),
  customer_id  BIGINT,
  event_type   STRING,
  payload      STRING
) USING DELTA
PARTITIONED BY (event_date);

-- Good: filter is on the partition column with constants
SELECT event_type, COUNT(*)
FROM events
WHERE event_date >= '2026-09-24' AND customer_id = 4711
GROUP BY event_type;

-- Bad: function wrapped around the partition column → no static pruning
SELECT COUNT(*) FROM events WHERE date_format(event_date,'yyyy-MM') = '2026-09';

-- Maintenance: compact small files and co-locate by customer_id
OPTIMIZE events WHERE event_date >= '2026-09-01' ZORDER BY (customer_id);
```

How to check it worked: `EXPLAIN FORMATTED` (or the Spark UI SQL tab) for the first query shows `PartitionFilters: [event_date >= 2026-09-24]` and `PushedFilters: [customer_id = 4711]`, and the scan node's metrics show *number of files read* and *size of files read* far below the table totals. In the second query `PartitionFilters: []` and all partitions are listed — a full scan. The rewrite `event_date BETWEEN '2026-09-01' AND '2026-09-30'` restores pruning. In Snowflake the equivalent evidence is `partitions scanned` vs `partitions total` in the query profile.

### 5. Under the hood
- **Statistics are only as good as the clustering.** If every file contains a random mix of customers, each file's `min=1, max=9,999,999`, and "skipping" skips nothing. Writing sorted data (or Z-ordering/clustering) makes ranges narrow and disjoint. Z-order interleaves bits of several columns so multi-column filters both get reasonable ranges; linear sort is best when one column dominates.
- **Overlap and depth.** Snowflake reports average *overlap* and *depth* (how many micro-partitions a given value range hits). High numbers mean data is poorly clustered; the cure is re-clustering (automatic reclustering costs credits) or a better clustering key, ideally a low-cardinality *expression* such as `date_trunc('day', ts)` rather than a raw high-cardinality timestamp.
- **Metadata is a bottleneck too.** A Hive metastore with millions of partitions makes *planning* slow (listing partitions, fetching their statistics) even before data is read. Table formats (Delta/Iceberg) move this into a log/manifests that are themselves pruned, which is one reason to use them.
- **Hidden partitioning (Iceberg).** The partition value is derived by a transform (`days(event_ts)`, `bucket(16, id)`), so queries filter on the real column and the engine maps it to partitions; you don't need a redundant `dt` column and can't forget to filter on it.
- **Bloom filters** help equality/`IN` on high-cardinality columns that can't be clustered; they are useless for ranges and cost write time and space. A false positive just means one wasted file read.
- **Files and parallelism.** Each file is at least one task. Few huge partitions limit parallelism; very many tiny ones make task scheduling and S3 `LIST`/`GET` latency dominate. Target roughly 128 MB–1 GB per file.
- **Updates.** Append-then-delete patterns fragment data into many small files and delete vectors; `OPTIMIZE`/compaction plus `VACUUM` of unreferenced files restores skipping.

### 6. Performance and design implications
- Choose partition columns that appear in **most filters** and have **moderate cardinality** (dates: yes; `user_id`: no). Aim for partitions of at least hundreds of MB.
- **Over-partitioning** (e.g. by hour *and* customer) → millions of tiny files, a flooded metastore, no benefit. **Under-partitioning** → no pruning, scans everything.
- Keep partition sizes roughly balanced to avoid skewed tasks.
- Partition by *time* usually also gives cheap retention (drop a partition instead of `DELETE`).
- Make predicates **sargable**: compare the raw column to constants; avoid `WHERE YEAR(dt)=2026`, `CAST`, or implicit conversions on the partition column. Where a derived filter is unavoidable, store the derived column (generated column) as the partition key.
- Project only the needed columns (`SELECT *` defeats columnar I/O savings).
- Cap runaway cost with `require_partition_filter` (BigQuery) or table-level guards; track bytes scanned.
- Tune `spark.sql.shuffle.partitions` and use broadcast joins for small dimensions (watch `autoBroadcastJoinThreshold`, or the "small" side OOMs the driver).
- Measure: files/partitions scanned vs total, bytes scanned, task count and size distribution, clustering depth.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `PARTITIONED BY (col)` / `PARTITION BY` | Defines physical partitioning | Table creation; pick filter-heavy, low-cardinality column |
| `OPTIMIZE t [WHERE …] ZORDER BY (c)` (Delta) | Compacts files, co-locates by column | After heavy small writes; for selective non-partition filters |
| `VACUUM t` (Delta) | Removes unreferenced files | Storage hygiene after compaction/deletes |
| `CALL system.rewrite_data_files(...)` (Iceberg) | Compaction / sort rewrite | Small-file cleanup, re-sorting |
| `ALTER TABLE … CLUSTER BY (expr)` / `SYSTEM$CLUSTERING_INFORMATION` | Set and inspect clustering (Snowflake) | Large tables with range/equality filters on a few columns |
| `CREATE BLOOMFILTER INDEX` (Databricks) | Per-file bloom filter | Equality/IN on high-cardinality, unclustered column |
| `spark.sql.optimizer.dynamicPartitionPruning.enabled` | Runtime partition filters | Star-schema joins with filtered dimension |
| `EXPLAIN FORMATTED` / Spark UI | Show `PartitionFilters`, `PushedFilters` | Verify pruning actually happened |
| `require_partition_filter` (BigQuery) | Reject queries without partition filter | Cost control |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| **Partition by date** — queries almost always filter by time range, data arrives in time order | Queries rarely filter by time, or daily volumes are only a few MB |
| **Z-order / clustering key** — selective filters on 1–3 columns that you can't partition on | Columns with extremely high cardinality used raw (timestamps, UUIDs) or tables with constant churn (re-clustering cost) |
| **Few larger partitions** — better file sizes, simpler metadata | Need fine-grained retention or very selective pruning |
| **Many small partitions** — precise pruning, fine retention | Small-file explosion; slow planning; low parallelism per partition |
| **Bucketing** — repeated joins/aggregations on same key | Bucket counts/columns differ between tables (shuffle returns), key skew |
| **Plain index/B-tree** (if your engine is an RDBMS) — point lookups and OLTP | Large analytical scans — partitioning/columnar layout wins |

### 9. Common mistakes
- **Partitioning by a high-cardinality column** (user_id) → millions of directories → metastore overload, tiny files → partition by date, bucket or Z-order the ID.
- **Wrapping the partition column in a function** → no pruning, full scan → filter the raw column or use a generated partition column.
- **Never compacting streaming/append output** → thousands of small files → scheduled `OPTIMIZE`/compaction and `VACUUM`.
- **Expecting statistics to skip on unclustered data** → file min/max cover the whole domain → sort/cluster at write time.
- **Bloom filters on every column** → write and storage overhead, false positives → only for selective equality filters.
- **Assuming partitioning is "an index"** → it only helps queries filtering on the partition key; other filters still scan the partition.
- **Ignoring skew** → one giant partition (e.g. `NULL` or 'unknown') stalls the job → salt or re-partition.
- **Setting shuffle partitions blindly** → tiny output files (too many) or spilling tasks (too few) → size to data volume or enable adaptive execution.

### 10. 🎯 Interview kit
**30-second answer:** "In lakes there are no B-trees; speed comes from reading less. Partitioning splits data into directories by a filter column so the planner prunes whole partitions, and Parquet/Delta/Iceberg store min/max per file so the engine skips files whose range can't match. Both need good layout — right-sized partitions, big files, and clustered/sorted data — otherwise you get the small-file problem or no skipping."

**Structure a longer answer:** clarify (engine, query patterns, data volume/arrival) → coarse tool (partition) → fine tool (stats/clustering/bloom) → maintenance (compaction, vacuum, re-clustering) → trade-offs (over-partitioning, write cost) → how you'd verify (plan, partitions scanned, bytes scanned).

**Likely follow-ups:**
- *Partitioning vs indexing?* → Partitioning prunes groups of data by a key you chose; an index finds individual rows. Partitioning is chosen at schema level and only helps filters on the key; indexes cost write overhead and storage.
- *Why do tiny files kill performance?* → per-file open/list overhead, weak statistics, too many tasks, metadata bloat.
- *How do you choose partition columns?* → filter frequency, cardinality, data size per partition, retention needs.
- *Filter on a derived column and pruning stops — what do you do?* → rewrite predicate on raw column, or materialize derived column as partition.
- *Snowflake has no indexes; what replaces them?* → micro-partition pruning on automatic metadata plus optional clustering keys.
- *Fewer large vs many small partitions?* → pruning granularity vs file size/parallelism/metadata cost.

**Signals interviewers look for:** you tie everything to *bytes scanned*, mention file sizing and compaction, know that statistics need clustering, and verify with the plan rather than assuming.

### 11. Check yourself
- [ ] Explain the difference between partition pruning and file-level data skipping with an example of each.
- [ ] Given a query log, pick a partition column and justify its cardinality and file sizes.
- [ ] Rewrite `WHERE YEAR(order_date) = 2026 AND MONTH(order_date) = 9` so pruning works.
- [ ] Describe what high overlap/depth means in Snowflake and how you would fix it.
- [ ] Explain how a small-file problem arises in a streaming job and how to prevent it.
- [ ] State when you'd add a bloom filter, and when you'd avoid it.
<!-- explainer:end -->

## Implementations (tools)

- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)
- [Hadoop](<../../../TOOLS/Hadoop/_index.md>)
- [Databricks](<../../../TOOLS/Databricks/_index.md>)

## Practice questions

- [Intermediate] [You load data into a partitioned Hive/Spark table by date. How is partitioning similar to and different from a traditional database index?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20275_You_load_data_into_a_partitioned_Hive_Spark_table_by_date_Ho.md>)
- [Intermediate] [When would you add a Databricks bloom filter index to a Delta table, and why is it only useful for certain predicate shapes?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20392_When_would_you_add_a_Databricks_bloom_filter_index_to_a_Delt.md>)
- [Intermediate] [How do you use Snowflake's QUERY_HISTORY / query profile to find a query with high partitions-scanned vs partitions-total, and what does that ratio tell you?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20440_How_do_you_use_Snowflake_s_QUERY_HISTORY_query_profile_to_fi.md>)
- [Deep] [Explain partition pruning versus predicate pushdown in Spark: are they the same optimization, and where does each happen in the plan?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20378_Explain_partition_pruning_versus_predicate_pushdown_in_Spark.md>)
- [Deep] [In BigQuery there are no indexes; explain how partitioning plus clustering give you pruning, and how they interact on a query.](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20397_In_BigQuery_there_are_no_indexes_explain_how_partitioning_pl.md>)
- [Deep] [A query filters on a low-cardinality partition column and a high-cardinality lookup column; how do you combine partitioning with clustering/Z-order for both?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20437_A_query_filters_on_a_low-cardinality_partition_column_and_a_.md>)
- [Deep] [Explain how a MERGE INTO on Delta rewrites entire files that contain matched rows, and how clustering the join key reduces the files touched.](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20461_Explain_how_a_MERGE_INTO_on_Delta_rewrites_entire_files_that.md>)
- [Expert] [Explain Iceberg's partition evolution: how can you change the partitioning of a huge table without rewriting old data, and how does pruning still work?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20389_Explain_Iceberg_s_partition_evolution_how_can_you_change_the.md>)

## Related concepts

- [Partitioning and Bucketing Design](<../../../02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Partitioning_and_Bucketing_Design.md>) *(Partitioning and Bucketing)*
- [Lakehouse Fundamentals](<../../../02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Lakehouse_Fundamentals.md>) *(Lakehouse)*
- [Distributed Joins in MPP Systems](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/_index.md>) *(Distributed Systems Core)*

## References

- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Chapter 6
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Chapter 6 › Chapter 9
- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Databricks Delta Lake › Learning about dynamic partition pruning
- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Databricks Delta Lake › Summary
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Scalability and Performance Optimization › 12. Scalability and Performance Optimization
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore metadata › Rowgroup Physical Metadata
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Bulk loading data › Columnstore Reorganize Operations with Each Data Load
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — partitioning › Database Maintenance Database Maintenance
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Index Maintenance › Reorganize to Remove Delta Rowgroups
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Diving into the Delta Lake Ecosystem › Table optimization
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Maintaining Your Delta Lake › Using OPTIMIZE to Fix the Small File Problem
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Performance Tuning Optimizing Your Data Pipelines with Delta Lake › Partition pruning and data skipping
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Performance Tuning Optimizing Your Data Pipelines with Delta Lake › Cluster By
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Storage Design Patterns › Problem
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Storage Design Patterns › Pattern: Sorter
- *Data Engineering Design Patterns* — Bartosz Konieczny — O › P
- *Data Engineering Design Patterns* — Bartosz Konieczny — R

