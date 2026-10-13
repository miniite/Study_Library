---
id: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Bounding_Memory_for_Skewed_Partitions"
level: 4
type: "concept"
topic: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design"
tools: ["Apache_Spark"]
---

# Bounding Memory for Skewed Partitions

**L4 Advanced** · [↑ Pipeline Design](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** When one partition is far bigger than the rest, a single worker has to hold or process it all, so you bound memory by splitting the hot key, streaming instead of materializing, and capping per-task work — rather than by buying a bigger machine.

### 1. The big picture
Distributed pipelines scale by cutting data into partitions and giving each partition to a worker. That only works if the partitions are roughly equal. Real data is not: one customer produces 40% of the events, one `NULL` key swallows every unmatched row, a single day (Black Friday) is ten times a normal day. The result is **skew**: most tasks finish in seconds while one task runs for an hour, runs out of memory, spills to disk, or gets killed and retried forever.

Two ideas from the pipeline books frame the problem. First, cloud compute is billed by the work done, not the wall-clock of one machine ("10 servers for 1 minute costs the same as 1 server for 10 minutes"), so the goal is *scalability and parallelism*, not squeezing speed out of one node. Skew is the enemy of that goal because it creates a serial tail. Second, optimization has limits: fixing skew adds complexity, so you do it where a measured bottleneck exists, keep a known-good fallback, and re-check as data volumes change.

This lesson is about the **memory** side: making sure that however lopsided the input, no single task needs more memory than you have allocated.

### 2. Core concepts and vocabulary
- **Partition** — the unit of data one task processes at a time (a file split, a Kafka partition, a hash bucket of a key).
- **Skew** — uneven partition sizes; the "hot key" or "hot partition" problem.
- **Straggler** — the one task that keeps the stage (and the job) from finishing.
- **Spill** — when an operator (sort, hash aggregate, join) runs out of memory and writes intermediate data to disk; correct but slow.
- **OOM** — out-of-memory kill; the task dies and is retried, usually dying again.
- **Salting** — appending a random suffix to a hot key so its rows spread over several partitions.
- **Streaming (iterator) processing** — handling records one at a time or in small chunks rather than loading the whole partition.
- **Backpressure** — a slow consumer signalling the producer to slow down so buffers stay bounded.
- **Concurrency limit** — a cap on how many pipeline runs or tasks execute at once, so combined memory demand stays bounded.

### 3. How it works, step by step
A practical approach is to work from "find it" to "contain it":

1. **Detect skew.** Look at per-task metrics: input size, duration, spill, peak memory. A healthy stage has max ≈ median × 1–2. If max is 10–100× the median, you have skew.
2. **Find the cause.** Usually one of: a hot key in a join/group-by, a `NULL` or default value key, a single huge file, a non-splittable compressed file (gzip), or time-based partitioning where one period dominates.
3. **Bound the memory per task** with the cheapest technique that fits:
   - **Process in chunks.** Read the partition as a stream/iterator, process N rows at a time, flush results. Memory is O(chunk), not O(partition).
   - **Repartition into more, smaller pieces** so each task's share shrinks (more partitions, or split by a secondary column such as an hour within the day).
   - **Salt the hot key** for joins/aggregations: aggregate in two stages (partial aggregate by `key + salt`, then final aggregate by `key`).
   - **Isolate the hot keys.** Handle the few giant keys through a separate path (e.g. broadcast the small side just for those keys) and the long tail through the normal path.
   - **Cap file/batch size at ingestion** so one input never exceeds a threshold; split oversize files on arrival.
4. **Bound the *concurrency* too.** A pipeline orchestrator can run several heavy runs at once (backfills especially). Set a concurrency limit or use workload management so that "max tasks × max memory per task" fits in the cluster; otherwise independent runs starve each other.
5. **Make failures cheap.** Keep tasks small and idempotent so the retry of one skewed slice does not redo the whole job (see the Checkpointing and Recovery lesson).
6. **Re-measure.** Skew moves: a fix today is irrelevant when a new hot customer appears. Alert on max/median task time.

### 4. Worked example
Spark job: join 2 billion click events to a customer table, then aggregate per customer. One customer has 600 million events.

```python
from pyspark.sql import functions as F

SALT_BUCKETS = 16

# 1. Spread the hot key across SALT_BUCKETS partitions on the big side
clicks_salted = clicks.withColumn(
    "salt", (F.rand() * SALT_BUCKETS).cast("int")
)

# 2. Partial aggregate on (customer_id, salt) -> bounded work per task
partial = (clicks_salted
    .groupBy("customer_id", "salt")
    .agg(F.count("*").alias("n"), F.sum("amount").alias("amt")))

# 3. Final aggregate collapses the salt back
final = (partial
    .groupBy("customer_id")
    .agg(F.sum("n").alias("clicks"), F.sum("amt").alias("amount")))
```

What happens: without salting, all 600M rows for the hot customer hash to one reducer, which must hold a hash-aggregate state and shuffle blocks for them; it spills heavily or OOMs. With 16 salt buckets, that customer is spread over 16 tasks of about 37M rows each; the second aggregation only handles 16 small rows per customer. In the Spark UI you would check the stage's task-duration histogram and "shuffle read size" column: the long tail should disappear. (Spark 3 AQE can also split skewed join partitions automatically — `spark.sql.adaptive.skewJoin.enabled` — which should be your first try.)

### 5. Under the hood   *(optional at L4)*
- **Why the straggler dominates:** a stage finishes when its slowest task finishes, so stage time = max task time, while cost = sum of task times. Skew inflates the first without changing the second much.
- **Memory is consumed by state, not by input.** A sort buffers rows; a hash aggregate or hash join holds a table per key group; a `collect_list` holds everything for a key. The dangerous operators are the ones whose state grows with the *key's* row count.
- **Spill is a safety valve with a price.** Spilling keeps the task alive but multiplies I/O; a task that spills 20× its memory can be slower than splitting the key.
- **Non-splittable inputs:** one gzip file cannot be read in parallel, so one task reads all of it regardless of cluster size. Use splittable formats (Parquet, bzip2, block-compressed) or split upstream.

### 6. Performance and design implications
- Measure: task duration min/median/max, shuffle read bytes per task, spill (memory/disk), GC time, executor OOM count.
- Cost: salting adds a second shuffle; chunked streaming trades some throughput for a flat memory curve. Both are usually cheaper than a larger instance type for the whole fleet.
- Target partition sizes around 100–256 MB for batch engines; much smaller partitions raise scheduling overhead, much larger raise OOM risk.
- Keep concurrency limits on shared orchestrators; one team's backfill should not starve everyone's daily jobs.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `spark.sql.adaptive.enabled` / `spark.sql.adaptive.skewJoin.enabled` | AQE splits oversized join partitions at runtime | First line of defense against skewed joins |
| `spark.sql.shuffle.partitions` | Number of shuffle partitions | Raise when partitions are too big; AQE coalescing can shrink the rest |
| `df.repartition(n, col)` | Redistribute rows by key or round-robin | Break up an oversized input partition |
| salt column + two-stage aggregation | Spread a hot key | Skewed group-by or join when AQE is not enough |
| Orchestrator concurrency / pool / workload management | Cap parallel runs | Protect shared clusters from backfill storms |
| Chunked read (`chunksize`, iterators) | Constant memory per task | Single-process Python/Pandas pipelines |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Streaming/chunked processing — memory must be flat and the logic is row-local or incrementally combinable | The operation needs the whole group at once (exact median, full sort) |
| Salting — a handful of hot keys dominate an aggregation or join | Skew is spread across many keys, or the extra shuffle costs more than it saves |
| Bigger executors/memory — a one-off job, quick fix | It becomes the permanent answer: the next hot key grows past it again |
| Isolating hot keys to a separate path — the hot set is small and known | The hot set changes daily and you cannot detect it automatically |

### 9. Common mistakes
- Raising executor memory first → hides skew, costs money, fails again later → find the hot key and fix distribution.
- Ignoring `NULL`/default keys → all unmatched rows land on one partition → filter or salt them explicitly.
- Increasing concurrency to "go faster" while a long task is the real bottleneck → no speed-up, more contention → fix the long tail first.
- Adding salt without a second aggregation step → wrong results (partial rows) → always recombine on the original key.
- Optimizing once and never re-checking → regressions when data patterns shift → alert on max/median task time.

### 10. 🎯 Interview kit
**30-second answer:** Skew means one partition carries disproportionate data, so one task OOMs or straggles. I confirm it from per-task metrics, then bound memory by repartitioning, salting hot keys with a two-stage aggregation, streaming in chunks, letting AQE split skewed joins, and capping concurrency — and I monitor so it does not regress.

**Structure a longer answer:** clarify (batch or stream, engine, what operator is failing) → detect with task metrics → contain with the cheapest fix → trade-offs (extra shuffle, complexity) → close with a real scenario (hot customer, NULL key).

**Likely follow-ups:**
- *How do you find the skewed key?* → group by the join/group key and count; look at top-N and `NULL`s; check Spark UI task distribution.
- *Salting vs broadcast join?* → broadcast if the other side is small; salting when both sides are large.
- *What if the input is one giant gzip file?* → it is non-splittable; split upstream or use a splittable codec/format.
- *How does this differ in streaming?* → state per key can be unbounded; use watermarks and state TTL, and fix hot Kafka partitions at the producer.

**Signals interviewers look for:** you measure before changing things, you know AQE exists, you distinguish memory limits from throughput, and you consider concurrency and retries, not only the single job.

### 11. Check yourself
- [ ] Explain why stage time is set by the slowest task, and how skew affects cost vs latency.
- [ ] Write a two-stage salted aggregation and say why the second stage is required.
- [ ] List three causes of skew other than "popular customer".
- [ ] Describe how you would prove from the Spark UI that a fix worked.
- [ ] Say when you would still choose to increase memory instead of redesigning.
<!-- explainer:end -->

## Implementations (tools)

- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)

## Practice questions

- [Intermediate] [Why would you break a large pipeline into smaller, independent tasks instead of one big script?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Q-12844_Why_would_you_break_a_large_pipeline_into_smaller_independen.md>)
- [Intermediate] [What happens if an API you are pulling from rate-limits your pipeline, and how do you handle it?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Q-12863_What_happens_if_an_API_you_are_pulling_from_rate-limits_your.md>)
- [Intermediate] [How would you compress large output files in a pipeline and what trade-offs does compression bring?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Q-12891_How_would_you_compress_large_output_files_in_a_pipeline_and_.md>)
- [Intermediate] [How would you decide the right batch size when loading data into a target database?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Q-12937_How_would_you_decide_the_right_batch_size_when_loading_data_.md>)
- [Intermediate] [What is a bottleneck in a pipeline and how would you go about finding it?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Q-12938_What_is_a_bottleneck_in_a_pipeline_and_how_would_you_go_abou.md>)
- [Intermediate] [How do you decide the right granularity for a task in an orchestrator — one big task vs many small tasks — and what are the failure-domain implications?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Q-12970_How_do_you_decide_the_right_granularity_for_a_task_in_an_orc.md>)
- [Deep] [How would you design a pipeline that processes data too large to fit in memory?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Q-12935_How_would_you_design_a_pipeline_that_processes_data_too_larg.md>)
- [Deep] [How do you decide between a micro-batch and a true streaming architecture when the business asks for 'real-time' but latency tolerance is actually a few minutes?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Q-12944_How_do_you_decide_between_a_micro-batch_and_a_true_streaming.md>)

## Related concepts

- [Pipeline Design Patterns](<Pipeline_Design_Patterns.md>) *(Pipeline Design)*
- [Capacity Planning and Throughput](<../../../01_Databases_and_SQL/Database_Operations/Capacity_Planning/Capacity_Planning_and_Throughput.md>) *(Capacity Planning)*

## References

- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — The Data Pipeline Is Not About Speed › Chapter 21. The Data Pipeline Is Not About Speed
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Small Files in a Big Data World A Data Engineer's Biggest Nightmare › Chapter 65. Small Files in a Big Data World: A Data Engineer's Biggest Nightmare
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Scalability and Performance Optimization › 12. Scalability and Performance Optimization
- *Data Engineering with Medallion Architecture* — Miki Eto — Building Scalable Data Pipelines › Process or steps
- *Data Engineering with Medallion Architecture* — Miki Eto — Building Scalable Data Pipelines › Cautions and limits
- *Data Pipelines Pocket Reference* — James Densmore — A Modern Data Infrastructure › Data Volume
- *Data Pipelines Pocket Reference* — James Densmore — Best Practices for Maintaining Pipelines › Scaling Complexity
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Cost Optimization for Data Pipelines
- *Data Engineering Best Practices* — Richard J. Schiller, David Larochelle — Detailed capabilities of the ingestion zones › How does big data processing complexity affect my data pipeline development?

