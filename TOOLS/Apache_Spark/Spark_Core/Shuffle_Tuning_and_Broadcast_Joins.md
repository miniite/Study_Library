---
id: "TOOLS/Apache_Spark/Spark_Core/Shuffle_Tuning_and_Broadcast_Joins"
level: 4
type: "concept"
topic: "TOOLS/Apache_Spark/Spark_Core"
tools: ["Apache_Spark"]
---

# Shuffle Tuning and Broadcast Joins

**L4 Advanced** · [↑ Spark Core](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Joins and aggregations force a shuffle; you control its cost by picking the join strategy (broadcast vs. sort-merge), sizing shuffle partitions, and letting Adaptive Query Execution coalesce partitions, switch strategies and split skew at runtime.

### 1. The big picture
This lesson is the *tuning* companion to the shuffle mechanics (see PySpark: Reducing Shuffle). Three levers matter:

1. **Join strategy** — whether the big table must be shuffled at all.
2. **Partition count and size** — how the shuffled data is divided into tasks.
3. **Adaptivity (AQE, Spark 3+)** — the engine re-optimises using actual runtime statistics: it reduces the number of reducers, converts sort-merge joins to broadcast joins, and handles skew in joins.

Everything is visible in `explain()` and the Spark UI, so tuning is "measure → change one lever → compare".

### 2. Core concepts and vocabulary
- **Partition** — unit of parallelism; tasks per stage = partitions. Check with `df.rdd.getNumPartitions()`.
- **Shuffle** — redistribution of rows by key between stages; costly because of disk + network.
- **Broadcast hash join (BHJ)** — small side collected to the driver and sent to every executor; no shuffle of the large side.
- **Sort-merge join (SMJ)** — both sides shuffled by key, sorted, merged; scalable default.
- **Shuffled hash join** — shuffle both, build a hash table of the smaller side per partition.
- **Cartesian / broadcast nested loop** — join without equality condition; O(n·m).
- **Skew** — a few keys hold most rows → a few huge tasks.
- **AQE** — `spark.sql.adaptive.enabled`; features: coalesce partitions, local shuffle reader, convert to BHJ, skew join.
- **Salting** — manually add a random suffix to hot keys to spread them.
- **Join hint** — `BROADCAST`, `MERGE`, `SHUFFLE_HASH`, `SHUFFLE_REPLICATE_NL`.
- **Dynamic partition pruning** — use the filtered dimension side to prune fact partitions at runtime.

### 3. How it works, step by step
Choosing a join strategy (Catalyst, simplified):
1. Is there an equality join condition? If not → broadcast nested loop or Cartesian (danger).
2. Is one side ≤ `spark.sql.autoBroadcastJoinThreshold` (default 10 MB) by *estimated* size, or hinted? → **BHJ**.
3. Else if sort-merge preferred (default) and keys sortable → **SMJ**: shuffle both by join key into `spark.sql.shuffle.partitions` partitions.
4. At runtime with AQE: after the shuffle map stage finishes, Spark knows real sizes — if one side is now small it replans as BHJ; small partitions are coalesced toward `advisoryPartitionSizeInBytes` (64 MB default); oversized skewed partitions are split into sub-partitions and the matching side replicated.

Sizing partitions: total shuffle bytes ÷ target 100–200 MB per partition, then at least 2–4× total cores.

### 4. Worked example
```python
spark.conf.set("spark.sql.adaptive.enabled", "true")
spark.conf.set("spark.sql.adaptive.coalescePartitions.enabled", "true")
spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")
spark.conf.set("spark.sql.shuffle.partitions", "2000")        # generous starting number; AQE coalesces

fact = spark.table("sales")              # 2 TB
dim  = spark.table("dim_store")          # 80 MB

q = fact.join(dim.hint("broadcast"), "store_id").groupBy("region").sum("amount")
q.explain("formatted")
```
Expected plan lines: `BroadcastHashJoin [store_id], [store_id], Inner, BuildRight` + `BroadcastExchange`; then `HashAggregate (partial)` → `Exchange hashpartitioning(region)` → `HashAggregate (final)`. With AQE the top node reads `AdaptiveSparkPlan isFinalPlan=true` after execution and shows `CustomShuffleReader coalesced` (or `AQEShuffleRead`) with the reduced partition count.

Skew check before running: `fact.groupBy("store_id").count().orderBy(F.desc("count")).show(10)` — one key with 100× the median is skew. Manual salting for a hot key when AQE isn't enough:
```python
N = 16
fact_s = fact.withColumn("salt", (F.rand()*N).cast("int"))
dim_s  = dim.crossJoin(spark.range(N).withColumnRenamed("id", "salt"))
fact_s.join(dim_s, ["store_id", "salt"])
```

### 5. Under the hood
- **BHJ mechanics**: executors send the build side to the driver, the driver builds a `HashedRelation` and broadcasts it (torrent broadcast); each executor probes it with its partition of the big side. Costs: driver memory + `spark.sql.broadcastTimeout` (300 s). A table that "should be small" but isn't (after expanding joins, stale/missing stats, a wide column) causes broadcast timeouts or driver OOM — check the plan's size estimates, run `ANALYZE TABLE ... COMPUTE STATISTICS`, or hint explicitly.
- **SMJ mechanics**: map tasks hash-partition rows by key and write one file per task; reducers fetch, sort (with spill to disk if memory is short) and merge-join. Skewed keys cause a single reducer to spill heavily or fail with `FetchFailedException`/OOM.
- **AQE skew join**: detects partitions larger than `skewedPartitionFactor` (5×) the median *and* `skewedPartitionThresholdInBytes` (256 MB), splits them into several tasks and reads the other side's matching partition multiple times. It only handles skew *in the join stage*, not in aggregations; for extreme skew salting or isolating the hot key is still needed.
- **Hint priority**: `BROADCAST` > `MERGE` > `SHUFFLE_HASH` > `SHUFFLE_REPLICATE_NL`; hints override statistics but cannot force an impossible strategy (e.g. broadcast of full outer side).
- **Dynamic partition pruning** (Spark 3): filters on the dimension are turned into a subquery that prunes fact partitions — works when the fact is partitioned by the join key.

### 6. Performance and design implications
- Raising `autoBroadcastJoinThreshold` without checking memory risks driver OOM rather than speed-ups.
- Too few shuffle partitions → big tasks, spill; too many → scheduler overhead (mitigated by AQE coalescing).
- Spill (memory/disk) columns in Stage UI are the first hint to add partitions or memory.
- Cache a DataFrame that feeds several joins instead of recomputing it.
- Use bucketed tables only when the same big-big join recurs; they remove the shuffle but cost write flexibility.
- Always compare runs by shuffle read/write bytes and stage duration (event log/History Server), not wall-clock alone.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `spark.sql.shuffle.partitions` | Reducer count (default 200) | Tune to data size |
| `spark.sql.autoBroadcastJoinThreshold` | Auto-broadcast limit (10 MB) | Raise carefully, `-1` disables |
| `spark.sql.broadcastTimeout` | Broadcast wait (300 s) | After verifying size |
| `spark.sql.adaptive.enabled` | AQE master switch | Spark 3+ (default on in 3.2+) |
| `spark.sql.adaptive.advisoryPartitionSizeInBytes` | Target post-shuffle partition size | Coalesce tuning |
| `spark.sql.adaptive.skewJoin.enabled` | Split skewed partitions | Join skew |
| `df.hint("broadcast")` / `/*+ BROADCAST(t) */` | Force strategy | Known small table |
| `ANALYZE TABLE t COMPUTE STATISTICS` | Collect stats | Better planning |
| `df.explain("formatted")` | Show plan | Always |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Broadcast: dimension ≪ executor memory, stable size | Table may grow; many concurrent big broadcasts |
| Sort-merge: large–large | One side tiny (waste) |
| AQE skew handling | Skew in aggregation, or extreme hot key (salt) |
| Salting | Simple cases where AQE works; adds complexity and a replicated side |
| Hint to override optimiser | Statistics are right — hints become stale as data grows |
| Fewer, bigger partitions | Memory pressure / spill |

### 9. Common mistakes
- Trusting default `200` partitions at any scale → mis-sized tasks → size by bytes, enable AQE.
- Forcing broadcast of a table that later grows → timeouts at 3 a.m. → monitor size or use AQE switch.
- Join without a condition (or with `OR`/non-equi) → Cartesian/BNLJ → rewrite equi-join, filter first.
- Repartitioning before a join "to be safe" → extra shuffle → only when it removes skew or enables co-location; benchmark with shuffle bytes.
- Ignoring unexpected `Exchange` nodes → silent shuffles → read the plan.

### 10. 🎯 Interview kit
**30-second answer:** "A shuffle moves data between stages; broadcast joins avoid it for small tables, sort-merge handles large-large. I size shuffle partitions by data volume, and rely on AQE to coalesce partitions, switch to broadcast at runtime and split skewed partitions — with salting as a fallback."

**Structure a longer answer:** what a shuffle costs → strategy selection → partition sizing → AQE → skew → verify via UI/event logs.

**Likely follow-ups:**
- *How does a broadcast join work internally?* → collect small side to the driver, broadcast hash table, probe locally.
- *Salting vs AQE?* → AQE is automatic, join-only and threshold-based; salting is manual, general, and costs a replicated dimension.
- *Cartesian product in the plan?* → missing/non-equi condition, fix join keys or pre-filter.
- *Spark 3 benefits out of the box vs. opt-in?* → AQE (on by default from 3.2), dynamic partition pruning, vs. configs/hints like broadcast thresholds and salting.
- *Prove a partitioning change helped?* → compare shuffle read/write bytes and stage times between two event logs in the History Server.

**Signals interviewers look for:** a measurement-based approach and knowledge of what AQE can't fix.

### 11. Check yourself
- [ ] Explain why BHJ avoids a shuffle and what can break it.
- [ ] Pick shuffle partition settings for 1 TB shuffled on 200 cores.
- [ ] Describe how AQE detects and splits a skewed partition.
- [ ] Write a salting join for a hot key.
<!-- explainer:end -->

## Implementations (tools)

- [Apache Spark](<../_index.md>)

## Practice questions

- [Basic] [What does it mean that RDDs are immutable and why is that useful?](<../../../PRACTICE/TOOLS/Apache_Spark/Spark_Core/Q-02245_What_does_it_mean_that_RDDs_are_immutable_and_why_is_that_us.md>)
- [Intermediate] [What is a shuffle in Spark and why is it expensive?](<../../../PRACTICE/TOOLS/Apache_Spark/Spark_Core/Q-02262_What_is_a_shuffle_in_Spark_and_why_is_it_expensive.md>)
- [Intermediate] [Your broadcast join is throwing a broadcast timeout on a table you expected to be small; how do you debug what actually got broadcast and what do you tune?](<../../../PRACTICE/TOOLS/Apache_Spark/Spark_Core/Q-02344_Your_broadcast_join_is_throwing_a_broadcast_timeout_on_a_tab.md>)
- [Deep] [How does Structured Streaming treat a stream as an unbounded table?](<../../../PRACTICE/TOOLS/Apache_Spark/Spark_Core/Q-02299_How_does_Structured_Streaming_treat_a_stream_as_an_unbounded.md>)
- [Deep] [How would you decide between salting a skewed join key versus enabling AQE skew join handling for a production pipeline?](<../../../PRACTICE/TOOLS/Apache_Spark/Spark_Core/Q-02343_How_would_you_decide_between_salting_a_skewed_join_key_versu.md>)
- [Deep] [When would you force a broadcast join with a hint, and what are the risks of overriding the optimizer this way?](<../../../PRACTICE/TOOLS/Apache_Spark/Spark_Core/Q-02358_When_would_you_force_a_broadcast_join_with_a_hint_and_what_a.md>)
- [Deep] [What performance benefits does Spark 3 give you out of the box, and which require you to opt in?](<../../../PRACTICE/TOOLS/Apache_Spark/Spark_Core/Q-02385_What_performance_benefits_does_Spark_3_give_you_out_of_the_b.md>)
- [Deep] [Walk me through why a query plan shows an unexpected Cartesian product and how you would fix it.](<../../../PRACTICE/TOOLS/Apache_Spark/Spark_Core/Q-02417_Walk_me_through_why_a_query_plan_shows_an_unexpected_Cartesi.md>)

## Related concepts

- [Shuffle Partition Tuning](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Shuffle_Partition_Tuning.md>) *(Big Data Processing)*
- [Spark Skew and Spill](<../Spark_Optimization/Spark_Skew_and_Spill.md>) *(Spark Optimization)*
- [Databricks Query Performance](<../../Databricks/Databricks_Platform/Databricks_Query_Performance.md>) *(Databricks Platform)*
- [Spark DataFrames and Lazy Evaluation](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Spark_DataFrames_and_Lazy_Evaluation.md>) *(Pipeline Orchestration)*

## References

- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Chapter 12. Epilogue: Apache Spark 3.0 › Adaptive Query Execution
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Chapter 12. Epilogue: Apache Spark 3.0 › The AQE framework
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — S
- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Big Data Analytics › Learning partitioning strategies in Spark
- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Big Data Analytics › Understanding Spark SQL optimizations
- *Hadoop: The Definitive Guide* — Tom White — Hive › Note
- *Databricks Certified Associate Developer for Apache Spark* — Saba Shah — What this book covers
- *Databricks Certified Associate Developer for Apache Spark* — Saba Shah — Data-based optimizations in Apache Spark › Managing data shuffle in Apache Spark
- *Databricks Certified Associate Developer for Apache Spark* — Saba Shah — What is Spark SQL › Performance and scalability
- *Data Engineering Design Patterns* — Bartosz Konieczny — Pattern: Dynamic Joiner › Examples
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Storage Design Patterns › Data producer
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Transforming Data with Apache Spark › Exploring Set Operations in Spark SQL
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Transforming Data with Apache Spark › Union operation

