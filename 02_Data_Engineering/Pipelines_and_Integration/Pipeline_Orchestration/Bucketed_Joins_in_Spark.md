---
id: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Bucketed_Joins_in_Spark"
level: 4
type: "concept"
topic: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration"
tools: ["Apache_Spark"]
---

# Bucketed Joins in Spark

**L4 Advanced** · [↑ Pipeline Orchestration](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A Spark join is expensive because it normally shuffles both sides across the network; broadcast joins avoid the shuffle for a small side, and bucketing pre-partitions and pre-sorts tables on the join key so a sort-merge join can skip the shuffle entirely.

### 1. The big picture
In Spark, joining two large tables requires all rows with the same key to meet on the same executor. By default Spark achieves that with a **shuffle**: both datasets are repartitioned by the join key and exchanged over the network, then sorted and merged. Shuffles are the dominant cost of most Spark jobs — network, disk spill, and skew all live there.

You have three levers: (1) avoid the shuffle by **broadcasting** the small side; (2) pay for the shuffle once by **bucketing** tables you join repeatedly; (3) tune the shuffle (partition counts, adaptive execution) and fix skew when neither of the others applies. The choice depends on table sizes, how often the join recurs, and whether you control how the tables are written.

### 2. Core concepts and vocabulary
- **Shuffle** — redistributing data across executors by key; writes shuffle files and reads them remotely.
- **Sort-merge join (SMJ)** — default for large-large equi-joins: shuffle both sides by key, sort each partition, merge.
- **Broadcast hash join (BHJ)** — ship the small table to every executor and join locally; no shuffle of the big side.
- **Shuffle hash join** — shuffle then build a hash table on the smaller side per partition.
- **Bucketing** — when writing a table, hash a column into a fixed number of buckets (and optionally sort within each) and store that layout in the metastore.
- **Partitioning (directory)** — splitting a table into folders by column value; helps pruning, different from bucketing.
- **`spark.sql.shuffle.partitions`** — number of partitions after a shuffle (default 200).
- **AQE (adaptive query execution)** — runtime re-optimization: coalesce partitions, switch join strategy, split skewed partitions.
- **Skew** — a few keys with many rows, making some tasks huge.

### 3. How it works, step by step
**Sort-merge join (no bucketing)**
1. Each side is scanned.
2. Rows are hashed on the join key into N shuffle partitions and written to shuffle files.
3. Each reducer fetches its partition from both sides over the network.
4. Each side is sorted by key within the partition.
5. The sorted streams are merged, emitting matches.

**Broadcast hash join**
1. The small side is collected to the driver (or built via a broadcast exchange).
2. It is sent to every executor and kept in memory as a hash table.
3. The big side is scanned and each row probes the hash table. No shuffle of the big side.
4. Applies only if the small side fits (`spark.sql.autoBroadcastJoinThreshold`, default 10 MB, often raised to hundreds of MB; or explicit hint).

**Bucketed sort-merge join**
1. Both tables were written earlier with `bucketBy(n, key)` using the *same* bucket count (or a multiple) and `sortBy(key)`.
2. At query time the planner sees matching bucket specs on the join key.
3. Bucket *i* of left is joined with bucket *i* of right directly — no exchange, and no sort if sorted within buckets.
4. The cost moved to write time.

### 4. Worked example
```python
# Write both tables once, bucketed and sorted by the join key
(events.write
   .bucketBy(64, "customer_id").sortBy("customer_id")
   .mode("overwrite").saveAsTable("dw.events_b"))

(customers.write
   .bucketBy(64, "customer_id").sortBy("customer_id")
   .mode("overwrite").saveAsTable("dw.customers_b"))

# Later: many joins reuse the layout
q = spark.table("dw.events_b").join(spark.table("dw.customers_b"), "customer_id")
q.explain()
```
Reading `explain()`: without bucketing you see `Exchange hashpartitioning(customer_id, 200)` and `Sort` under each side of `SortMergeJoin`. With matching buckets those `Exchange` nodes disappear. For a small dimension you would instead see `BroadcastHashJoin` with a `BroadcastExchange` on the small side; force it with `broadcast(customers)`. In the Spark UI, compare shuffle read/write bytes for the stage before and after.

### 5. Under the hood   *(optional at L4)*
- **Why bucketing works:** bucket id = hash(key) mod N is deterministic, so both tables agree on where a key lives; the planner's "output partitioning" metadata lets it elide exchanges.
- **Constraints:** bucket counts must be compatible; the join key must be the bucketing column(s); the layout is recorded in the table metadata, so it only applies to catalog tables written through Spark's bucketing, not to arbitrary files. Some table formats (Delta, Iceberg) handle layout differently — Iceberg has bucket transforms, Delta commonly uses partitioning plus Z-order/liquid clustering instead.
- **Broadcast failure modes:** a "small" side that grows past executor memory causes OOM or driver bottleneck; AQE can convert SMJ to BHJ at runtime when post-filter size is small.
- **Skew:** a key with huge row counts overloads one bucket/partition; AQE's skew-join handling splits it, or salting is required.
- **Partition count:** too few → big tasks and spills; too many → scheduling overhead and tiny files. AQE coalescing adjusts post-shuffle partitions to a target size.

### 6. Performance and design implications
- Broadcast beats bucketing when one side is small: no layout work needed.
- Bucketing pays off for large-large joins repeated many times; writing is slower and requires discipline (consistent counts).
- Over-partitioned tables (high-cardinality partition column) create many small files and slow planning; bucket instead or choose a coarser partition.
- Measure with the Spark UI: shuffle read/write, task time distribution, spill, and skewed tasks (one task reading 90% of data).
- Filter and project early to shrink inputs before joins; this can make a broadcast possible.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `broadcast(df)` / `/*+ BROADCAST(t) */` | Force broadcast join | Small dimension, known size |
| `spark.sql.autoBroadcastJoinThreshold` | Auto-broadcast size limit | Tune for cluster memory; `-1` disables |
| `df.write.bucketBy(n, col).sortBy(col)` | Create bucketed table | Large tables repeatedly joined |
| `spark.sql.shuffle.partitions` | Shuffle parallelism | Set explicitly if AQE is off |
| `spark.sql.adaptive.enabled` | Enable AQE (coalesce, skew, join switch) | Default on in recent Spark; keep on |
| `spark.sql.adaptive.skewJoin.enabled` | Split skewed partitions | Skewed key joins |
| `df.explain("formatted")` | Show physical plan | Check for Exchange/Sort nodes |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Broadcast join — one side is small after filters | Side may grow unpredictably; memory risk |
| Sort-merge join — both sides large, one-off | Same join repeats often — consider bucketing |
| Bucketed tables — stable large fact/dim joined repeatedly on same key | Many different join keys; bucket count hard to keep consistent |
| AQE + more partitions — data volume varies by day | You need deterministic plans for debugging |
| Salting — severe skew AQE can't fix | Skew is mild; extra shuffle not worth it |

### 9. Common mistakes
- Broadcasting a table that is "usually small" → OOM during incident spikes → cap with threshold and monitor size.
- Bucketing with mismatched bucket counts → shuffle returns silently → verify with `explain`.
- Partitioning by a high-cardinality column → millions of small files → partition by date/coarse column, bucket by key.
- Leaving `shuffle.partitions` at 200 for terabytes → huge tasks/spill → raise it or rely on AQE.
- Ignoring skew because the average looks fine → one straggler → inspect task max vs median.

### 10. 🎯 Interview kit
**30-second answer:** Spark joins normally shuffle both sides by key then sort-merge. If one side is small I broadcast it to avoid the shuffle. If two large tables are joined repeatedly on the same key I bucket and sort them on that key at write time so the join needs no exchange. I verify with `explain` and the Spark UI and use AQE or salting for skew.

**Structure a longer answer:** clarify sizes and frequency → default SMJ cost → broadcast → bucketing → skew/partition tuning → measure.

**Likely follow-ups:**
- *Broadcast vs sort-merge?* → broadcast when small side fits; otherwise SMJ.
- *Bucketing vs partitioning?* → partitioning = folders for pruning; bucketing = hash layout for joins/aggregations.
- *What if broadcast side is too large?* → executor/driver OOM or slow collect; fall back to SMJ.
- *How set shuffle partitions?* → aim for ~100–200 MB per partition; use AQE coalescing.

**Signals interviewers look for:** you reason from the shuffle cost, read plans, and know the operational limits of each strategy.

### 11. Check yourself
- [ ] List the steps and costs of a sort-merge join.
- [ ] State two conditions for a bucketed join to skip the shuffle.
- [ ] Say how you would detect a skewed join key in the Spark UI.
- [ ] Choose a join strategy for a 2 TB fact table and a 50 MB dimension.
<!-- explainer:end -->

## Implementations (tools)

- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)

## Practice questions

- [Intermediate] [What is a shuffle in Spark and why is it considered an expensive operation?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13361_What_is_a_shuffle_in_Spark_and_why_is_it_considered_an_expen.md>)
- [Deep] [How would you tune bucketing or Z-ordering on the enriched-alert table to speed up the service-level joins your correlation training does most?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13516_How_would_you_tune_bucketing_or_Z-ordering_on_the_enriched-a.md>)
- [Deep] [How do you decide the partitioning strategy for a large event table so both time-range scans and per-customer lookups stay efficient without over-partitioning?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13667_How_do_you_decide_the_partitioning_strategy_for_a_large_even.md>)
- [Deep] [How would you cut the cloud cost of a Spark pipeline that scans full tables daily by adopting partition pruning predicate pushdown and column pruning?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13670_How_would_you_cut_the_cloud_cost_of_a_Spark_pipeline_that_sc.md>)
- [Deep] [Explain the insert-overwrite-by-partition pattern in Spark and how it makes daily loads safely re-runnable versus a blind append.](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13691_Explain_the_insert-overwrite-by-partition_pattern_in_Spark_a.md>)
- [Deep] [What is salting a skewed join key in Spark and what are the correctness caveats you must handle when un-salting the result?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13693_What_is_salting_a_skewed_join_key_in_Spark_and_what_are_the_.md>)
- [Deep] [How do you decide between broadcasting a dimension table and using a bucketed join in Spark when the dimension is on the edge of the broadcast threshold?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13720_How_do_you_decide_between_broadcasting_a_dimension_table_and.md>)
- [Deep] [Explain when you would choose broadcast join over sort-merge join in Spark for a feature pipeline and what happens if the broadcast side is too large.](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13143_Explain_when_you_would_choose_broadcast_join_over_sort-merge.md>)

## Related concepts

- [Spark Skew and Spill](<../../../TOOLS/Apache_Spark/Spark_Optimization/Spark_Skew_and_Spill.md>) *(Spark Optimization)*
- [Shuffle Partition Tuning](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Shuffle_Partition_Tuning.md>) *(Big Data Processing)*

## References

- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Optimizing and Tuning Spark Applications › Maximizing Spark parallelism
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Optimizing and Tuning Spark Applications › A Family of Spark Joins
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Optimizing and Tuning Spark Applications › When to use a broadcast hash join
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Optimizing and Tuning Spark Applications › Optimizing the shuffle sort merge join
- *Data Engineering Fundamentals* — Zhaolong Liu — CHAPTER 8Monitoring and Performance › CHAPTER 8Monitoring and Performance(#contents.xhtmlch08a)
- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Big Data Analytics › Understanding bucketing in Spark
- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Spark Core › Dynamically optimizing skew joins
- *Designing Data-Intensive Applications* — Martin Kleppmann — Stream Joins
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Transforming Data with Apache Spark › Intersect operation
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Storage Design Patterns › Broadcasting Big Tables

