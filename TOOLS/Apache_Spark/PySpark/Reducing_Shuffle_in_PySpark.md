---
id: "TOOLS/Apache_Spark/PySpark/Reducing_Shuffle_in_PySpark"
level: 4
type: "concept"
topic: "TOOLS/Apache_Spark/PySpark"
tools: ["Apache_Spark"]
---

# Reducing Shuffle in PySpark

**L4 Advanced** · [↑ PySpark](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A shuffle is Spark moving rows across the network so that equal keys meet; you reduce it by pre-aggregating, broadcasting small tables, choosing the right partition count and API, and letting AQE fix what static planning can't.

### 1. The big picture
Spark's speed comes from narrow transformations (`filter`, `select`, `map`) that run inside one partition with no data movement. A **wide transformation** (`groupBy`, `join`, `distinct`, `repartition`, window with `partitionBy`) needs rows with the same key on the same partition, so Spark writes intermediate data to local disk, ships it over the network, and reads it back: a **shuffle**. Each shuffle is also a **stage boundary**, so the number of shuffles drives the number of stages, the amount of disk/network I/O, and most of the runtime of a typical job.

"Reducing shuffle" therefore means three things: (1) avoid the shuffle entirely (broadcast join, pre-partitioned data), (2) shuffle less data (filter early, select only needed columns, partial aggregation), and (3) shuffle sensibly (right number and size of partitions, no skew).

### 2. Core concepts and vocabulary
- **Partition** — a slice of a DataFrame processed by one task; number of tasks in a stage = number of partitions.
- **Narrow vs. wide dependency** — narrow: each output partition depends on one input partition; wide: depends on many (needs shuffle).
- **Shuffle write / shuffle read** — map-side output files written locally, then fetched by reduce-side tasks.
- **Map-side combine** — partial aggregation before the shuffle (`reduceByKey`, DataFrame `groupBy().agg()`); `groupByKey` on RDDs does not combine.
- **`spark.sql.shuffle.partitions`** — number of partitions after a shuffle (default 200).
- **Broadcast join** — ship the small table to every executor so the big table is never shuffled.
- **AQE (Adaptive Query Execution)** — Spark 3 re-plans at runtime using shuffle statistics: coalesces small post-shuffle partitions, switches sort-merge to broadcast join, splits skewed partitions.
- **Exchange** — the operator name in `explain()` that marks a shuffle.
- **`repartition` vs `coalesce`** — repartition does a full shuffle to any count; coalesce only merges existing partitions (no shuffle, can only decrease).

### 3. How it works, step by step
What happens on `df.groupBy("user").count()`:
1. **Stage 1 (map side):** each task reads its input partition, does a *partial* aggregation (per-task counts), hashes the key to pick one of N shuffle partitions, and writes N sorted/partitioned files to local disk.
2. **Shuffle service:** reduce tasks fetch their slice from every map task over the network.
3. **Stage 2 (reduce side):** each task merges its incoming partial counts and emits final rows.
4. If AQE is on, after stage 1 Spark sees the real sizes of the N partitions and may merge many tiny ones into fewer tasks.

Ways to cut cost, in rough order of payoff:
1. Filter and project early (also enables predicate pushdown / column pruning).
2. Broadcast the small side of a join.
3. Use built-in aggregations (partial aggregation) rather than collecting groups.
4. Dedup on narrow key sets; avoid `distinct` on wide rows.
5. Size shuffle partitions to ~100–300 MB each.
6. Co-locate: write both tables bucketed by the join key, or reuse the same partitioning for consecutive operations.
7. Enable AQE and skew handling.

### 4. Worked example
```python
from pyspark.sql import functions as F

# Bad: shuffles the full wide fact table, then joins on the shuffled side
res = (facts                                    # 2 TB, 60 columns
       .join(dim, "country_id")                 # dim is 80 MB -> still a sort-merge join if stats are missing
       .groupBy("country_name")
       .agg(F.sum("amount").alias("revenue")))

# Better: prune, broadcast, and aggregate before/while joining
res = (facts.select("country_id", "amount")     # column pruning
            .where("event_date >= '2026-10-01'")  # predicate pushdown
            .join(F.broadcast(dim.select("country_id", "country_name")), "country_id")
            .groupBy("country_name")
            .agg(F.sum("amount").alias("revenue")))

res.explain("formatted")   # look for BroadcastHashJoin and a single Exchange hashpartitioning(country_name)
```
Reading the plan: `BroadcastHashJoin` with `BroadcastExchange` means the dimension is broadcast and the fact table was not shuffled for the join. The remaining `Exchange hashpartitioning(country_name, 200)` is the aggregation shuffle — it moves only *partial sums* (one row per country per task), not 2 TB. In the Spark UI, SQL tab → look at "shuffle bytes written" per exchange to confirm.

### 5. Under the hood
- **Why `groupByKey` is worse than `reduceByKey`:** `reduceByKey` combines values per key inside each map task before writing shuffle files; `groupByKey` ships every record and builds the full list per key on the reducer (memory risk). DataFrames avoid this because Catalyst plans a partial aggregate + final aggregate.
- **200 default:** a 5 TB stage on 200 partitions is 25 GB per task → spill and OOM; a 50 MB job on 200 partitions yields 200 tiny tasks of scheduling overhead. Rule of thumb: partitions ≈ shuffle bytes / 128–200 MB, at least 2–4× total cores. With AQE `coalescePartitions` the high initial number is acceptable because Spark merges small ones.
- **Join strategies:** broadcast hash (no shuffle, build side must fit in executor/driver memory), shuffled hash, sort-merge (both sides shuffled and sorted; the default for large-large), broadcast nested loop / Cartesian (no equi-condition — catastrophic; check for missing join predicates in the plan).
- **`coalesce` pitfalls:** it keeps upstream parallelism low — a `coalesce(1)` after a heavy computation forces the *whole* upstream stage to run in one task; use `repartition(n)` when the upstream work is expensive, `coalesce` when only reducing output files after a cheap filter.
- **AQE** needs statistics from completed shuffle stages, so it re-optimises only at stage boundaries; it can convert sort-merge → broadcast if one side turns out small *after* filters.

### 6. Performance and design implications
- Measure with the Spark UI: Stages tab → "Shuffle Read/Write size", "Spill (memory/disk)", task duration quantiles (max ≫ median = skew).
- `spark.sql.autoBroadcastJoinThreshold` (default 10 MB): raising it too high risks driver/executor OOM because the table is first collected to the driver.
- Shuffle files live on executor local disk: lost executors cause `FetchFailedException` and stage retries (external shuffle service / disaggregated shuffle mitigates).
- Fewer, larger output files: coalesce *after* the last shuffle or use AQE; use partitioned writes with a sensible partition column to avoid "50,000 tiny files".
- Python UDFs don't change shuffle count but force serialisation; prefer built-ins or pandas (Arrow) UDFs.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `df.explain("formatted")` | Show physical plan; find `Exchange` | Counting shuffles |
| `spark.sql.shuffle.partitions` | Post-shuffle partition count | Tune for data volume |
| `spark.sql.adaptive.enabled` (+ `coalescePartitions`, `skewJoin`) | Runtime re-planning | Default on in recent Spark |
| `F.broadcast(df)` / `/*+ BROADCAST(t) */` | Force broadcast join | Small dimension known up front |
| `spark.sql.autoBroadcastJoinThreshold` | Auto-broadcast size limit | Raise carefully / `-1` to disable |
| `df.repartition(n, "col")` | Full shuffle by key | Before heavy join/agg, or to even out skew |
| `df.coalesce(n)` | Merge partitions, no shuffle | Reduce output files cheaply |
| `df.rdd.getNumPartitions()` | Inspect partition count | Debugging |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Broadcast join — one side < ~100s of MB | Table grows over time → broadcast timeout / OOM |
| `repartition` before join | Data is already well partitioned (extra shuffle for nothing) |
| `coalesce` to cut files | Upstream computation is heavy (kills parallelism) |
| Raise shuffle partitions | Many small tasks; with AQE prefer a high start + coalescing |
| Bucketing | Few distinct joins; small tables; write cost and rigidity |

### 9. Common mistakes
- Leaving `shuffle.partitions=200` for both tiny and multi-TB jobs → overhead or spills → size by data volume, use AQE.
- `coalesce(1)` for a single output file on large data → one task does everything → write many files, compact later.
- Calling `count()` then `collect()` on an uncached plan → the whole DAG runs twice → cache/persist reused DataFrames.
- Forgetting a join condition → Cartesian/BroadcastNestedLoopJoin → read the plan, add the predicate.
- Using `groupByKey` or `collect_list` then reducing in Python → huge shuffle and memory → use built-in aggregations.

### 10. 🎯 Interview kit
**30-second answer:** "A shuffle redistributes rows by key across the network, writing and reading intermediate files, so it's the main cost and the stage boundary. I reduce it by filtering and pruning early, broadcasting small tables, using partial aggregation, sizing shuffle partitions sensibly and enabling AQE."

**Structure a longer answer:** define narrow/wide → how to find shuffles (`explain`, UI) → eliminate (broadcast, co-partition) → shrink (prune, pre-aggregate) → tune (partitions, AQE) → skew → verify with metrics.

**Likely follow-ups:**
- *Why is 200 partitions bad?* → too few for big data (spill), too many for small (overhead); target 100–300 MB per partition.
- *repartition vs coalesce?* → full shuffle, any count vs. narrow merge, decrease only.
- *Broadcast timeout?* → the "small" table isn't small after joins/filters; check plan statistics, raise `spark.sql.broadcastTimeout` only after confirming size, or fall back to sort-merge.
- *Job slowed 20 min → 3 h after doubled data?* → look for skew (max task ≫ median), spill, or too few partitions per stage.
- *dropDuplicates vs window dedup?* → both shuffle by key; `dropDuplicates(keys)` carries less data than window + `row_number` over wide rows.

**Signals interviewers look for:** you read `explain` and the UI instead of guessing; you can name the join strategies; you know AQE's limits.

### 11. Check yourself
- [ ] Mark the shuffles and stages in a `join → groupBy → orderBy` plan.
- [ ] Explain what `reduceByKey` does on the map side that `groupByKey` does not.
- [ ] Choose a shuffle partition count for 600 GB of shuffle data on 100 cores.
- [ ] Describe two ways to turn a sort-merge join into a broadcast join and the risks.
<!-- explainer:end -->

## Implementations (tools)

- [Apache Spark](<../_index.md>)

## Practice questions

- [Basic] [What is the difference between a narrow and a wide transformation and why does that distinction determine your job's number of stages?](<../../../PRACTICE/TOOLS/Apache_Spark/PySpark/Q-36466_What_is_the_difference_between_a_narrow_and_a_wide_transform.md>)
- [Intermediate] [Explain how a broadcast join works internally and how you'd decide the threshold for broadcasting a dimension table in production.](<../../../PRACTICE/TOOLS/Apache_Spark/PySpark/Q-36444_Explain_how_a_broadcast_join_works_internally_and_how_you_d_.md>)
- [Deep] [Your write step produces 50000 tiny Parquet files per day — walk me through the small-files problem and how you'd fix it upstream in the Spark job.](<../../../PRACTICE/TOOLS/Apache_Spark/PySpark/Q-36409_Your_write_step_produces_50000_tiny_Parquet_files_per_day_wa.md>)
- [Deep] [Describe how you'd implement an idempotent PySpark batch write to a partitioned table so reruns don't create duplicates.](<../../../PRACTICE/TOOLS/Apache_Spark/PySpark/Q-36418_Describe_how_you_d_implement_an_idempotent_PySpark_batch_wri.md>)
- [Deep] [Walk me through how a window function with partitionBy and orderBy executes physically and why an unbounded partition can blow up memory.](<../../../PRACTICE/TOOLS/Apache_Spark/PySpark/Q-36425_Walk_me_through_how_a_window_function_with_partitionBy_and_o.md>)
- [Deep] [How do you decide between partitioning bucketing and Z-ordering when writing a large fact table with PySpark for downstream query performance?](<../../../PRACTICE/TOOLS/Apache_Spark/PySpark/Q-36453_How_do_you_decide_between_partitioning_bucketing_and_Z-order.md>)
- [Deep] [How does Adaptive Query Execution (AQE) change a plan at runtime and which three optimizations does it apply — when have you seen it help or hurt?](<../../../PRACTICE/TOOLS/Apache_Spark/PySpark/Q-36477_How_does_Adaptive_Query_Execution_AQE_change_a_plan_at_runti.md>)
- [Deep] [How do you handle wide transformations that produce more partitions than executor cores can process — is more partitions always better?](<../../../PRACTICE/TOOLS/Apache_Spark/PySpark/Q-36496_How_do_you_handle_wide_transformations_that_produce_more_par.md>)

## Related concepts

- [Spark Skew and Spill](<../Spark_Optimization/Spark_Skew_and_Spill.md>) *(Spark Optimization)*
- [Bucketed Joins in Spark](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Bucketed_Joins_in_Spark.md>) *(Pipeline Orchestration)*

## References

- *Data Algorithms with Spark* — Mahmoud Parsian — **Serverless Spark** › CHAPTER 2
- *Data Algorithms with Spark* — Mahmoud Parsian — **CHAPTER 10** › CHAPTER 10
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Introduction to Apache Spark A Unified Analytics Engine › In Python
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Spark SQL and DataFrames Introduction to Built-in Data Sources › In Python
- *Databricks Certified Associate Developer for Apache Spark* — Saba Shah — Repartition a DataFrame into 8 partitions › Repartition a DataFrame into 8 partitions\
- *Databricks Certified Associate Developer for Apache Spark* — Saba Shah — Coalesce a DataFrame to 4 partitions › Coalesce a DataFrame to 4 partitions\
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Basic Operations on Delta Tables › Perform the GROUP BY, average (AVG), HAVING and order by equivalents\

