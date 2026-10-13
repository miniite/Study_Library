---
id: "TOOLS/Databricks/Databricks_Platform/Databricks_Query_Performance"
level: 4
type: "concept"
topic: "TOOLS/Databricks/Databricks_Platform"
tools: ["Databricks", "Apache_Spark"]
---

# Databricks Query Performance

**L4 Advanced** · [↑ Databricks Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Databricks query speed comes from reading less (file skipping via partitioning/clustering/statistics), moving less (avoiding or shrinking shuffles with broadcast joins and AQE), and keeping Spark's work evenly spread (no skew, right partition count, healthy file sizes).

### 1. The big picture
A Spark job on Databricks is slow for a small set of reasons: it **reads too much data**, it **shuffles too much data**, it **has uneven work** (skew), or it **runs through slow code paths** (Python UDFs, row-by-row logic). Delta Lake adds a fifth: **too many small files**. Tuning is the process of identifying which of these dominates using the Spark UI and query profile, then applying the matching lever.

This lesson connects Spark fundamentals (transformations, actions, shuffles) to the Databricks-specific tools: Delta data skipping, `OPTIMIZE`/liquid clustering, Adaptive Query Execution, Photon and caching.

### 2. Core concepts and vocabulary
- **Transformation vs action** — transformations (`select`, `filter`, `join`) build a lazy plan; actions (`count`, `write`, `collect`) trigger execution.
- **Narrow vs wide transformation** — narrow works within a partition; wide (join, `groupBy`, `distinct`, `repartition`) needs a **shuffle**.
- **Shuffle** — redistributing data across the network by key; involves serialization, disk and network I/O. The main cost driver in most jobs.
- **Broadcast join** — send a small table to every executor so the big side is not shuffled.
- **Partition** — unit of parallelism; one task per partition.
- **`repartition(n)`** — full shuffle to exactly *n* balanced partitions; **`coalesce(n)`** — merges to fewer partitions without a full shuffle (cannot increase).
- **Data skipping** — Delta uses per-file min/max statistics to avoid reading files that can't match a filter.
- **`OPTIMIZE … ZORDER BY`** — compacts files and co-locates related values; **liquid clustering** — newer incremental clustering that replaces partitioning + Z-order and can be changed without rewriting the table.
- **AQE (Adaptive Query Execution)** — runtime re-planning: coalesce shuffle partitions, switch to broadcast joins, split skewed partitions.
- **Photon** — native vectorized engine for SQL/DataFrame operations.
- **Spill** — when a task's data doesn't fit in memory it writes to disk; slow and visible in the Spark UI.
- **`cache()` / `persist()`** — keep a DataFrame in memory/disk for reuse.

### 3. How it works, step by step
A systematic tuning loop:

1. **Find the slow stage** — Spark UI / query profile: which stage dominates duration? Look at input size, shuffle read/write, spill and task-time distribution.
2. **Classify the problem**
   - Large *input* with a selective filter → data layout (partitioning, clustering, statistics).
   - Large *shuffle read/write* → join strategy, aggregation design, partition count.
   - A few tasks far slower than the median → skew.
   - Huge number of tiny tasks / files → small-file problem.
   - Low CPU, slow Python → UDFs or driver-side work.
3. **Apply the matching lever** (below) and re-measure on the same data.
4. **Verify** — compare runtime *and* shuffle/spill/files-read metrics, not just wall clock.

### 4. Worked example
```sql
-- A dashboard filters the events table by customer and a date range
SELECT customer_id, sum(amount)
FROM main.silver.events
WHERE event_date BETWEEN '2026-09-01' AND '2026-09-30' AND customer_id = 4711
GROUP BY customer_id;
```
Symptom in the query profile: *files read = 38,000 of 40,000*. The table is partitioned by `event_date` only, so the `customer_id` filter can't prune files.

Fix:
```sql
ALTER TABLE main.silver.events CLUSTER BY (event_date, customer_id);  -- liquid clustering
OPTIMIZE main.silver.events;                                          -- cluster existing data
```
After re-running: files read drops to a few hundred, because per-file min/max ranges for `customer_id` are now tight. If liquid clustering is unavailable on the runtime, `OPTIMIZE … ZORDER BY (customer_id)` gives a similar effect.

**Join strategy**
```python
from pyspark.sql.functions import broadcast
result = fact.join(broadcast(dim_customer), "customer_id")   # dim fits in memory
```
Spark broadcasts automatically below `spark.sql.autoBroadcastJoinThreshold` (10 MB default); AQE can convert a sort-merge join to a broadcast at runtime when one side turns out small after filtering.

### 5. Under the hood
- **Why shuffles are expensive** — map side writes shuffle files partitioned by key to local disk; reduce side fetches its slice from every map task over the network. Cost grows with data volume and with *number of partitions × number of map tasks* (file count).
- **`spark.sql.shuffle.partitions`** — default 200. Too many tiny partitions waste scheduling; too few cause spill. With AQE (`spark.sql.adaptive.enabled`) coalescing handles the "too many" side automatically; Databricks also offers `auto` shuffle partitions.
- **Join algorithms** — broadcast hash join (no shuffle of big side), shuffle hash join, sort-merge join (default for large/large), cartesian. Bucketing both sides on the join key removes the shuffle for repeated joins but requires write-time discipline.
- **Skew** — one key with disproportionate rows makes one task process most data. AQE's skew-join handling splits it; manual remedies are salting the key or isolating the hot key.
- **Delta statistics** — only the first 32 columns collect min/max by default; put frequently filtered columns early or change `delta.dataSkippingNumIndexedCols`.
- **`distinct()` vs `dropDuplicates(cols)`** — `distinct()` deduplicates on all columns; `dropDuplicates(["a","b"])` on the listed columns (keeping an arbitrary row per key — use a window with `row_number()` if you need *which* row).
- **Photon** — executes supported operators in vectorized native code on CPU; unsupported operators fall back to regular Spark. It is not GPU-based.

### 6. Performance and design implications
- **Partition sparingly**: choose low-cardinality columns that are almost always filtered; target ≥ 1 GB of data per partition. Over-partitioning is worse than under-partitioning.
- **Cluster instead**: for multi-column or high-cardinality filters use liquid clustering or Z-order; re-evaluate after workload changes.
- **Right-size files**: `OPTIMIZE` toward 128 MB–1 GB; enable optimized writes / auto compaction for frequent small appends.
- **Filter early, project early**: select only needed columns (Parquet is columnar) and push filters before joins.
- **Avoid Python UDFs** where a built-in function exists; use pandas UDFs (Arrow) if you must.
- **Cache only when reused**: caching a DataFrame used once costs memory and gains nothing; unpersist when done. Delta disk cache on workers speeds repeated reads automatically.
- **After a "partitioning change made the dashboard 10× slower"**: compare files-scanned and bytes-read before/after with `EXPLAIN`/query profile; if the new partition column isn't in the dashboard's filters, pruning is lost — add clustering on the filtered columns or revert, rather than re-modelling blindly.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `EXPLAIN FORMATTED` / query profile | show plan and metrics | start of any tuning |
| `OPTIMIZE t ZORDER BY (c)` | compact + co-locate | selective filters on non-partition columns |
| `ALTER TABLE t CLUSTER BY (…)` | liquid clustering | replace partition+zorder |
| `broadcast(df)` hint | force broadcast join | small dimension tables |
| `spark.sql.adaptive.enabled` | AQE on/off (on by default) | leave on |
| `spark.sql.shuffle.partitions` | shuffle width | when not relying on AQE |
| `repartition(n, col)` / `coalesce(n)` | control partition count | before writes, after heavy filters |
| `df.cache()` / `unpersist()` | reuse a result | iterative or multi-use data |
| `ANALYZE TABLE … COMPUTE STATISTICS` | refresh optimizer stats | poor join choices |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Broadcast the dimension — small enough to fit comfortably in executor memory | the "small" table is hundreds of MB or grows; OOM and driver pressure |
| Bucket both sides — a very large join repeats daily on the same key | ad-hoc joins; bucketing needs consistent writers and costs write time |
| `repartition` — need even sizes/skew relief before a write or join | just reducing file count (use `coalesce` or optimized writes) |
| `coalesce` — shrink partitions after a selective filter | needing more partitions or balanced sizes |
| Liquid clustering — unpredictable multi-column filters | tiny tables where layout doesn't matter |
| Photon — SQL/ETL heavy on scans, joins, aggregates | Python-UDF-bound work or very short jobs where DBU premium isn't recovered |

### 9. Common mistakes
- Tuning without reading the Spark UI → changing configs blindly → find the dominant stage first.
- `repartition` where `coalesce` suffices → extra full shuffle → use `coalesce` to reduce.
- Broadcasting a table that is actually large → executor OOM → check size after filters, rely on thresholds/AQE.
- Partitioning by a high-cardinality column → small files and slow listings → cluster instead.
- Treating `cache()` as free → memory eviction and slower jobs → cache only reused data.
- Calling `count()` repeatedly to "check" → recomputes the whole plan each time → persist or use `limit`.
- Using `distinct()` when only key columns define duplicates → wrong semantic and wasted work → `dropDuplicates(keys)` or window dedupe.

### 10. 🎯 Interview kit
**30-second answer:** Look at the Spark UI to see whether the job is bound by input, shuffle, skew or small files. Reduce input with partitioning/clustering and data skipping, reduce shuffle with broadcast joins and AQE, fix skew by splitting or salting hot keys, and compact files with `OPTIMIZE`.

**Structure a longer answer:** clarify symptom and baseline → locate the stage → classify the cause → apply and measure one change at a time → close with a before/after metric (files read, shuffle bytes, runtime, DBUs).

**Likely follow-ups:**
- *What is a shuffle and why is it slow?* → all-to-all repartition by key: serialization, disk, network; cost scales with data and partition count.
- *repartition vs coalesce?* → full shuffle and balanced vs narrow merge and possibly uneven.
- *Broadcast vs bucketing?* → broadcast for small dimensions with no setup; bucketing for repeated big-big joins with write-time cost.
- *What does AQE do?* → coalesces partitions, switches join strategy, handles skew using runtime statistics.
- *When does Photon not help?* → Python UDFs, small data, I/O-bound or driver-bound jobs.

**Signals interviewers look for:** you measure first, can name the metric that proves each fix, and weigh cost (DBUs) against speed.

### 11. Check yourself
- [ ] Given a query profile, say whether the problem is layout, shuffle, skew or small files.
- [ ] Explain how AQE changes a sort-merge join at runtime.
- [ ] Choose between partitioning, Z-order and liquid clustering for a workload.
- [ ] Rewrite a `groupByKey`-style job to avoid an unnecessary shuffle.
- [ ] Describe how you'd prove a tuning change helped.
<!-- explainer:end -->

## Implementations (tools)

- [Databricks](<../_index.md>)
- [Apache Spark](<../../Apache_Spark/_index.md>)

## Practice questions

- [Basic] [What is a Spark DataFrame and how do you create one from a CSV file in Databricks?](<../../../PRACTICE/TOOLS/Databricks/Databricks_Platform/Q-21772_What_is_a_Spark_DataFrame_and_how_do_you_create_one_from_a_C.md>)
- [Basic] [How would you remove duplicate rows from a DataFrame and what is the difference between distinct and dropDuplicates?](<../../../PRACTICE/TOOLS/Databricks/Databricks_Platform/Q-21816_How_would_you_remove_duplicate_rows_from_a_DataFrame_and_wha.md>)
- [Intermediate] [What is the difference between overwrite and append save modes when writing a table and when would you use each?](<../../../PRACTICE/TOOLS/Databricks/Databricks_Platform/Q-21778_What_is_the_difference_between_overwrite_and_append_save_mod.md>)
- [Intermediate] [What does explode() do in Spark and when would you use it on an array column?](<../../../PRACTICE/TOOLS/Databricks/Databricks_Platform/Q-21812_What_does_explode_do_in_Spark_and_when_would_you_use_it_on_a.md>)
- [Deep] [What is a shuffle in Spark and why can it make a job slow?](<../../../PRACTICE/TOOLS/Databricks/Databricks_Platform/Q-21781_What_is_a_shuffle_in_Spark_and_why_can_it_make_a_job_slow.md>)
- [Deep] [What is the difference between repartition and coalesce and when would you use each?](<../../../PRACTICE/TOOLS/Databricks/Databricks_Platform/Q-21822_What_is_the_difference_between_repartition_and_coalesce_and_.md>)
- [Deep] [A downstream BI dashboard on Databricks SQL got 10x slower after a partitioning change. How do you decide whether to add liquid clustering, revert, or re-model?](<../../../PRACTICE/TOOLS/Databricks/Databricks_Platform/Q-21869_A_downstream_BI_dashboard_on_Databricks_SQL_got_10x_slower_a.md>)
- [Deep] [How do you decide between broadcasting a dimension and bucketing both sides of a recurring large join?](<../../../PRACTICE/TOOLS/Databricks/Databricks_Platform/Q-21905_How_do_you_decide_between_broadcasting_a_dimension_and_bucke.md>)

## Related concepts

- [Pandas vs Polars vs PySpark](<../../Python/Python_Language/Pandas_vs_Polars_vs_PySpark.md>) *(Python Language)*
- [Spark DataFrames and Lazy Evaluation](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Spark_DataFrames_and_Lazy_Evaluation.md>) *(Pipeline Orchestration)*
- [DataFrames and Datasets](<../../Apache_Spark/Spark_Core/DataFrames_and_Datasets.md>) *(Spark Core)*
- [Spark Skew and Spill](<../../Apache_Spark/Spark_Optimization/Spark_Skew_and_Spill.md>) *(Spark Optimization)*
- [Bucketed Joins in Spark](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Bucketed_Joins_in_Spark.md>) *(Pipeline Orchestration)*
- [Shuffle Partition Tuning](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Shuffle_Partition_Tuning.md>) *(Big Data Processing)*

## References

- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Discovering Databricks › Chapter 1: Discovering Databricks
- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Learning about Machine Learning and Graph Processing in Databricks › Practicing ML in Databricks
- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Big Data Analytics › Understanding the use of inferSchema
- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Big Data Analytics › Summary
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Getting Started with Databricks › Apache Spark™ on Databricks
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Mastering Relational Entities in Databricks › Temporary views
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Transforming Data with Apache Spark › Querying Using the text Format
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Transforming Data with Apache Spark › Registering Tables from Files with CTAS
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Apache Spark's Structured APIs › Key Merits and Benefits
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Apache Spark's Structured APIs › The Catalyst Optimizer
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Apache Spark's Structured APIs › Phase 1: Analysis
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Apache Spark's Structured APIs › Phase 3: Physical planning
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — handling. Instead of failing immediately on a bad input line (or worse, failing silently and
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Performance Tuning Optimizing Your Data Pipelines with Delta Lake › Optimized writes
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Performance Tuning Optimizing Your Data Pipelines with Delta Lake › Example
- *Data Engineering Design Patterns* — Bartosz Konieczny — Error Management Design Patterns › Examples
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ordering › Examples
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Storage Design Patterns › Examples
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Quality Design Patterns › Examples

