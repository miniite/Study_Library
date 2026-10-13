---
id: "TOOLS/Apache_Spark/Spark_Optimization/Spark_Skew_and_Spill"
level: 4
type: "concept"
topic: "TOOLS/Apache_Spark/Spark_Optimization"
tools: ["Apache_Spark"]
---

# Spark Skew and Spill

**L4 Advanced** · [↑ Spark Optimization](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Skew (a few partitions much larger than the rest) and spill (data that doesn't fit in execution memory and goes to disk) are the two main reasons a Spark stage is slow or dies; you find them in task-level metrics and fix them with better partitioning, key handling and memory sizing.

### 1. The big picture
A stage finishes only when its *slowest* task finishes. If 199 tasks take 20 s and one takes 40 min, the job takes 40 min and 199 cores sit idle. That is **skew**. When a task's data (sorted/aggregated/joined) exceeds its share of execution memory, Spark writes intermediate runs to disk and merges later: **spill** — slower but not fatal. When even that fails, the task OOMs or the shuffle fetch fails (`FetchFailedException`).

Skew and spill are related: skew creates oversized partitions, which are exactly the ones that spill. Too-few partitions create the same effect without skew. So diagnosis starts with the same question: *what does the distribution of partition sizes and task durations look like?*

### 2. Core concepts and vocabulary
- **Skew** — uneven data per partition, usually from a hot key (`NULL`, `'unknown'`, a mega-customer).
- **Straggler task** — a task much slower than the median in its stage.
- **Spill (memory / disk)** — Spark UI metrics: in-memory size of spilled data and its on-disk (serialized) size.
- **Execution vs. storage memory** — regions of the unified memory pool.
- **Target partition size** — roughly 100–300 MB per task is a practical sweet spot.
- **Salting** — add a random suffix to a hot key to spread rows; the other side is replicated.
- **Two-phase aggregation** — aggregate by `(key, salt)` first, then by `key`.
- **Small-files problem** — too many tiny output files (poor write partitioning), hurting later reads.
- **Local aggregator** — design where records of a key are already co-located, so no shuffle is needed.
- **`FetchFailedException`** — a reducer couldn't fetch map output (lost executor, overloaded shuffle server, oversized blocks).

### 3. How it works, step by step
Diagnosis routine:
1. **Spark UI → Stages → the slow stage**: look at the summary metrics table (min/25th/median/75th/max) for duration, shuffle read size, spill. Max ≫ median ⇒ skew; uniformly large with spill ⇒ too few/too big partitions.
2. **Identify the key**: `df.groupBy(key).count().orderBy(desc("count")).show()`; approximate with `approx_count_distinct` or a sample for cheap first look; or `df.rdd.glom().map(len).collect()` for partition sizes.
3. **Read the plan**: which `Exchange` feeds that stage (join key? aggregation key?).
4. **Choose the fix** based on cause:
   - Few huge partitions because of too few shuffle partitions → increase `spark.sql.shuffle.partitions` or enable AQE coalescing/splitting.
   - Skewed *join* → AQE skew-join, broadcast the small side, or salting.
   - Skewed *aggregation* → two-phase (salted) aggregation, pre-aggregate map-side, `approx_*` functions.
   - Null/default hot key → filter it out of the join, process it separately.
   - Executor memory too small → raise memory, lower cores per executor, reduce row width.
5. **Re-run and compare** shuffle read max/median and spill totals.

### 4. Worked example
```python
from pyspark.sql import functions as F

# 1) Confirm skew
(events.groupBy("customer_id").count()
       .orderBy(F.desc("count")).show(5))
# customer_id=NULL -> 410M rows, next best 3M -> hot key

# 2) Separate the hot key, join the rest normally
hot  = events.where(F.col("customer_id").isNull())
rest = events.where(F.col("customer_id").isNotNull()).join(F.broadcast(dim_small), "customer_id")

# 3) Two-phase aggregation for a skewed groupBy
N = 32
salted = events.withColumn("salt", (F.rand() * N).cast("int"))
partial = salted.groupBy("customer_id", "salt").agg(F.sum("amount").alias("amt"))
final   = partial.groupBy("customer_id").agg(F.sum("amt").alias("amount"))

# 4) Prove the improvement from the UI/event log: max shuffle read per task, spill (disk), duration
```
What to look at after the change: the stage's task-time max should approach the median; "Spill (Disk)" should drop to ~0; the number of tasks processing the hot key goes from 1 to N. Salting adds a second shuffle, so total work rises slightly while wall-clock falls dramatically.

### 5. Under the hood
- **Spill mechanics**: during a sort-merge join or hash aggregation, tasks accumulate records in an in-memory structure (sorter/hash map). When it exceeds the task's execution memory share (~ pool ÷ active tasks), it is sorted and written as a spill file; at the end spills are merge-read. Hence **more cores per executor ⇒ less memory per task ⇒ more spill**.
- **Shuffle block limit**: a single reducer block > 2 GB historically caused failures; skewed keys hit this limit. Splitting the partition (AQE or salting) fixes it.
- **AQE** coalesces many *small* shuffle partitions up to `advisoryPartitionSizeInBytes` and splits *large skewed* ones in joins. It doesn't fix skew in `groupBy` or in window functions with a hot `partitionBy` key — for these, salting or restructuring (e.g., running totals per customer done in time buckets then combined) is needed.
- **`groupByKey`/`collect_list`** build the whole group in memory → OOM on hot keys; built-in aggregates use partial (map-side) aggregation and are bounded.
- **`dropDuplicates` / `distinct`** hash the whole row set by key → shuffle volume ∝ row width; project to key columns first, or use window `row_number` only when you need to choose the winner.
- **Broadcast-nested-loop** from a missing equi-condition is a different "slow" — O(n·m) CPU, not spill; fix the predicate.
- **Local aggregator**: if the producer writes each key to a fixed partition (e.g., Kafka key → same partition and Spark reads that partitioning), aggregations can be done without a shuffle, but Spark won't know it unless the plan encodes it — design on the producer side.

### 6. Performance and design implications
- Sizing rule: partitions ≥ 2–4 × cores and ~100–300 MB each; `shuffle.partitions` ≈ shuffle bytes ÷ 200 MB.
- Compaction: after heavy filters, `coalesce` before writing; target file sizes 128 MB–1 GB depending on engine and read pattern (bigger = fewer metadata ops, but less parallelism and more cost to rewrite).
- Executor layout: fewer cores/executor or larger memory when spilling; watch GC time.
- Track over time: stage duration variance, max/median task ratio, spill bytes, shuffle read/write, output file counts, cluster-hours vs. SLA.
- Cheaply detect skew *before* a run: approximate counts on the join key from a sample or metadata.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| Spark UI Stage page → summary metrics | Min/median/max task metrics | First diagnosis step |
| `df.rdd.glom().map(len).collect()` | Partition sizes | Empirical skew check (small data/sample) |
| `spark.sql.adaptive.skewJoin.enabled` | Split skewed join partitions | Join skew |
| `spark.sql.adaptive.coalescePartitions.enabled` | Merge small partitions | Post-shuffle sizing |
| `spark.sql.shuffle.partitions` | Reducer count | Size to volume |
| `spark.executor.memory` / `.cores` / `memoryOverhead` | Per-task memory | Spill/OOM |
| `repartition(n, col)` / `coalesce(n)` | Control partitioning | Even out / reduce files |
| `approx_count_distinct`, `F.broadcast` | Cheap distinct; avoid shuffle | Huge cardinality; small dimension |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| AQE skew-join | Skew in aggregations/windows |
| Salting | Simple skew handled by AQE; extra shuffle and complexity |
| Isolate hot key | Hot key set changes constantly |
| More partitions | Tiny tasks dominate (overhead); fix with AQE coalescing |
| More memory per executor | GC pauses, cost; fix data layout first |
| `repartition` before join | Already well partitioned — pays an extra shuffle |

### 9. Common mistakes
- Treating spill as an error → it's a signal; fix partition size or memory only if it dominates runtime.
- Setting `shuffle.partitions=200` and processing 5 TB → 25 GB tasks → size properly.
- `coalesce` after a filter that kept 10 % of rows without checking upstream parallelism → fewer tasks for the whole stage; consider `repartition` if upstream is heavy.
- Using `coalesce(1)` for a single file → serial bottleneck and OOM → write many files, compact separately.
- Salting both sides identically instead of replicating the dimension → wrong join results.

### 10. 🎯 Interview kit
**30-second answer:** "I look at the stage's task metrics: if max ≫ median it's skew, if everything is big with disk spill the partitions are too large or memory too small. Fixes are AQE skew handling, salting or isolating hot keys, two-phase aggregation, right-sizing shuffle partitions and executor memory."

**Structure a longer answer:** symptoms in UI → locate key and operator → classify (skew vs. too few partitions vs. memory) → targeted fix → verify with metrics.

**Likely follow-ups:**
- *Why does a hot key cause OOM and not just slowness?* → one task must hold its whole group/sort buffer; spill thresholds and 2 GB block limits are hit.
- *Running total per customer without a giant partition?* → bucket by (customer, time-bucket), compute partial sums, then combine carry-over values.
- *FetchFailedException root causes?* → executor loss/OOM, overloaded shuffle service, huge blocks from skew, network timeouts.
- *Ideal file size?* → balance metadata/listing overhead against parallelism and rewrite cost; commonly 128 MB–1 GB.
- *Benchmark repartition before a join?* → compare shuffle bytes, max task time and total cluster-time with and without.

**Signals interviewers look for:** quantitative diagnosis, not folklore; knowing AQE's boundaries.

### 11. Check yourself
- [ ] Read a stage summary and decide skew vs. under-partitioning.
- [ ] Implement salted two-phase aggregation correctly.
- [ ] Explain why more executor cores can increase spill.
- [ ] Describe how AQE's skew-join splits a partition and what it does to the other side.
<!-- explainer:end -->

## Implementations (tools)

- [Apache Spark](<../_index.md>)

## Practice questions

- [Basic] [Explain the difference between narrow and wide dependencies with a concrete example, and why only wide ones incur a shuffle stage boundary.](<../../../PRACTICE/TOOLS/Apache_Spark/Spark_Optimization/Q-39695_Explain_the_difference_between_narrow_and_wide_dependencies_.md>)
- [Intermediate] [How do you decide the ideal target file size (128MB vs 512MB vs 1GB) when compacting output, considering downstream read and metadata cost?](<../../../PRACTICE/TOOLS/Apache_Spark/Spark_Optimization/Q-39675_How_do_you_decide_the_ideal_target_file_size_128MB_vs_512MB_.md>)
- [Intermediate] [Explain why a dropDuplicates on a wide dataset triggers a shuffle and how you'd minimize the data shuffled for dedup.](<../../../PRACTICE/TOOLS/Apache_Spark/Spark_Optimization/Q-39737_Explain_why_a_dropDuplicates_on_a_wide_dataset_triggers_a_sh.md>)
- [Deep] [You suspect a wide transformation is the cost driver. How do you read the DAG to count shuffle boundaries and estimate data movement between stages?](<../../../PRACTICE/TOOLS/Apache_Spark/Spark_Optimization/Q-39660_You_suspect_a_wide_transformation_is_the_cost_driver_How_do_.md>)
- [Deep] [A job's shuffle-heavy stage keeps failing with FetchFailedException. Walk me through the root causes and your recovery/tuning path.](<../../../PRACTICE/TOOLS/Apache_Spark/Spark_Optimization/Q-39681_A_job_s_shuffle-heavy_stage_keeps_failing_with_FetchFailedEx.md>)
- [Deep] [How would you benchmark whether repartitioning before a join actually pays for its own shuffle cost? What metrics decide it?](<../../../PRACTICE/TOOLS/Apache_Spark/Spark_Optimization/Q-39701_How_would_you_benchmark_whether_repartitioning_before_a_join.md>)
- [Deep] [Explain how a broadcast-nested-loop join arises and why it can be catastrophically slow. How do you force a better plan?](<../../../PRACTICE/TOOLS/Apache_Spark/Spark_Optimization/Q-39714_Explain_how_a_broadcast-nested-loop_join_arises_and_why_it_c.md>)
- [Deep] [How would you use the Spark event log / history server to compare two runs and prove a partitioning change reduced shuffle bytes?](<../../../PRACTICE/TOOLS/Apache_Spark/Spark_Optimization/Q-39736_How_would_you_use_the_Spark_event_log_history_server_to_comp.md>)

## Related concepts

- [Bucketed Joins in Spark](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Bucketed_Joins_in_Spark.md>) *(Pipeline Orchestration)*
- [Join Strategies in Spark](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Join_Strategies_in_Spark.md>) *(Distributed Joins)*
- [Distributed Joins in MPP Systems](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/_index.md>) *(Distributed Systems Core)*
- [Shuffle Tuning and Broadcast Joins](<../Spark_Core/Shuffle_Tuning_and_Broadcast_Joins.md>) *(Spark Core)*
- [Shuffle Partition Tuning](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Shuffle_Partition_Tuning.md>) *(Big Data Processing)*
- [Reducing Shuffle in PySpark](<../PySpark/Reducing_Shuffle_in_PySpark.md>) *(PySpark)*

## References

- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Batch Processing Patterns › 6. Batch Processing Patterns
- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Databricks Delta Lake › Understanding optimized writes
- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Spark Core › Chapter 7: Spark Core
- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Spark Core › Understanding shuffle partitions
- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Spark Core › Dynamically coalescing shuffle partitions
- *Data Algorithms with Spark* — Mahmoud Parsian — **Serverless Spark** › CHAPTER 2
- *Data Algorithms with Spark* — Mahmoud Parsian — **CHAPTER 3** › CHAPTER 3
- *Data Algorithms with Spark* — Mahmoud Parsian — **CHAPTER 4** › CHAPTER 4
- *Data Algorithms with Spark* — Mahmoud Parsian — **PART II** › PART II
- *Mapping Data Flows in Azure Data Factory* — Mark Kromer — Monitor, Manage, and Optimize › Partitioning Strategies
- *Hadoop: The Definitive Guide* — Tom White — Spark › Note
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Optimizing and Tuning Spark Applications › Configuring Spark executors' memory and the shuffle service
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — D
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — E › G

