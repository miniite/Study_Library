---
id: "TOOLS/Apache_Spark/Spark_Core/Spark_Execution_Caching_and_Debugging"
level: 4
type: "concept"
topic: "TOOLS/Apache_Spark/Spark_Core"
tools: ["Apache_Spark", "Hadoop"]
---

# Spark Execution Caching and Debugging

**L4 Advanced** · [↑ Spark Core](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Spark turns your code into jobs → stages → tasks that run on executors under a driver; understanding that model lets you cache the right things, size resources, and debug a slow or failing job from the UI and the plan.

### 1. The big picture
A Spark application is one **driver** process (holds your program, builds the plan, schedules tasks) plus many **executors** (JVMs on worker nodes that run tasks and hold cached data), obtained from a **cluster manager** (YARN, Kubernetes, standalone, or Databricks). You write transformations; nothing runs until an **action**. Each action creates a **job**, split at shuffle boundaries into **stages**, each stage made of one **task** per partition.

Compared with MapReduce, Spark keeps intermediate data in memory/local disk between stages in the same job and has a DAG scheduler, which is why iterative and multi-step pipelines are much faster. This lesson covers the execution model, caching and checkpointing, and a systematic debugging routine.

### 2. Core concepts and vocabulary
- **Driver / executor / cluster manager** — coordinator / workers / resource allocator.
- **SparkSession** — unified entry point (wraps the older `SparkContext`).
- **Application, job, stage, task** — whole program; one per action; split at shuffles; one per partition.
- **DAG** — directed acyclic graph of operations; used for scheduling and lineage recovery.
- **Deploy mode** — *client* (driver on the submitting machine; logs local) vs. *cluster* (driver inside the cluster).
- **Executor memory / cores / overhead** — heap (`spark.executor.memory`), parallel tasks per executor (`spark.executor.cores`), off-heap & Python (`spark.executor.memoryOverhead`).
- **`cache()` / `persist(level)`** — keep a DataFrame's computed partitions for reuse.
- **Storage level** — `MEMORY_ONLY`, `MEMORY_AND_DISK` (DataFrame default), `DISK_ONLY`, serialized/replicated variants.
- **Checkpoint** — write data to reliable storage and *truncate lineage*.
- **Broadcast variable / accumulator** — read-only shared value / write-only counters (debug metrics).
- **Speculation, retries** — re-run slow or failed tasks.

### 3. How it works, step by step
1. `spark-submit` starts the driver (client or cluster mode) and requests executors.
2. Your code builds a logical plan; Catalyst optimises it.
3. An **action** (`count`, `write`, `collect`) submits a **job**.
4. The DAG scheduler cuts the plan at shuffle boundaries into **stages**; stages run in dependency order.
5. Each stage's **tasks** (one per partition) are sent to executors; tasks read input or shuffle files, compute, and write output or shuffle files.
6. Failed tasks are retried (default 4 attempts); lost shuffle output triggers re-running the map stage (`FetchFailedException`).
7. Results return to the driver (small) or are written to storage (large).
8. If a DataFrame is cached, later jobs read it from executor memory/disk instead of recomputing from source.

### 4. Worked example
```python
from pyspark import StorageLevel

df = (spark.read.parquet("s3://lake/silver/events/")
        .where("event_date >= '2026-10-01'")
        .join(dim, "user_id"))                    # expensive

df.persist(StorageLevel.MEMORY_AND_DISK)          # mark; nothing computed yet
n   = df.count()                                  # job 1: computes AND fills the cache
top = df.groupBy("country").count().collect()     # job 2: reads from cache
df.write.mode("overwrite").parquet("s3://lake/gold/events_enriched/")  # job 3
df.unpersist()                                    # free executor memory
```
How to inspect: Spark UI → **Jobs** (three jobs), **Stages** (shuffle read/write, task time distribution, spill), **Storage** (the cached RDD, fraction cached, size in memory vs. disk), **Executors** (GC time, failed tasks), **SQL** (the plan with metrics). Without `persist`, jobs 2 and 3 would each re-read and re-join from scratch. `df.count()` followed by `df.collect()` on an uncached DataFrame executes the lineage twice.

### 5. Under the hood
- **Cache is lazy and best-effort**: it's populated by the first action that touches *all* partitions (`count()` does; `take(1)` doesn't). Partitions that don't fit in `MEMORY_ONLY` are simply recomputed; `MEMORY_AND_DISK` spills them. Cached blocks are lost if an executor dies and are recomputed from lineage — caching is not fault tolerance.
- **Checkpoint vs. cache**: checkpoint writes to reliable storage (HDFS/S3) and cuts the lineage, so long iterative jobs don't build huge plans and recovery doesn't replay from the start; it costs a full write. Use `localCheckpoint` for speed with less safety.
- **Memory model** (unified): executor heap is split into execution (shuffles, joins, sorts) and storage (cache) regions that borrow from each other; user code gets the rest. Container size = heap + overhead; YARN/K8s kill containers that exceed it ("Container killed… exceeding memory limits") — typically fix by raising `memoryOverhead`, especially with PySpark or large broadcasts, or by reducing cores per executor.
- **Python**: in PySpark, driver-side code is Python but the plan runs in the JVM; Python workers appear only for UDFs/RDD lambdas. Use Arrow-based pandas UDFs to cut serialisation cost.
- **Driver risks**: `collect()` and huge broadcast tables materialise on the driver and cause OOM or `maxResultSize` errors.
- **Scheduling**: `spark.default.parallelism` (RDD) vs. `spark.sql.shuffle.partitions` (SQL); tasks per executor = cores; dynamic allocation scales executors with backlog.

### 6. Performance and design implications
- Cache when a DataFrame is used ≥ 2 times *and* is expensive to recompute *and* fits; otherwise caching costs memory for nothing.
- Prefer `MEMORY_AND_DISK` for reused large frames; `MEMORY_ONLY` risks silent recomputation, serialized levels trade CPU for space.
- Typical executor sizing: 4–5 cores and 16–32 GB heap per executor; avoid huge JVM heaps (GC pauses) and single-core executors (broadcast duplication).
- Monitor over time: job duration, input/shuffle bytes, spill, GC time, failed task counts, executor loss; alert on gradual regressions (compare against previous runs).
- Fail fast on data quality: use a cheap aggregated check (`F.count`, null rate) on the batch and raise before writing to the warehouse.
- Testing: keep transformations as pure functions `DataFrame -> DataFrame`, run with `local[2]` and small in-memory inputs in CI; mock I/O.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `df.cache()`, `df.persist(level)`, `unpersist()` | Reuse computed data | Reused expensive DataFrames |
| `df.checkpoint()` / `spark.sparkContext.setCheckpointDir` | Cut lineage, reliable save | Iterative jobs, very long plans |
| `spark.executor.memory`, `.cores`, `.memoryOverhead` | Executor sizing | OOM/killed containers |
| `spark.driver.memory`, `spark.driver.maxResultSize` | Driver capacity | Large collects/broadcasts |
| `--deploy-mode client|cluster` | Where the driver runs | Interactive vs. production |
| `spark.dynamicAllocation.enabled` | Elastic executors | Shared clusters |
| `df.explain(True)` | Parsed/analysed/optimised/physical plans | Debugging |
| `spark.eventLog.enabled` + History Server | Post-mortem UI | Comparing runs |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `MEMORY_AND_DISK` cache | DataFrame used once (waste) |
| `checkpoint` | Short lineage; extra write not worth it |
| Few big executors | Heavy GC; failures lose lots of state |
| Many small executors | Broadcast and overhead duplication |
| Client mode | Production long-running jobs (driver tied to your machine) |
| Local mode | Big data — single machine only, good for tests |

### 9. Common mistakes
- Caching everything → evicts useful data and slows GC → cache selectively and `unpersist`.
- Assuming `cache()` computes immediately → first action pays → trigger with `count()` if needed.
- `collect()` large DataFrames → driver OOM → aggregate or write out; use `toLocalIterator` if you must iterate.
- Raising `executor.memory` for container-killed errors when `memoryOverhead` was the real limit → still killed.
- Debugging by guessing → read the SQL plan and stage task-time distribution first.

### 10. 🎯 Interview kit
**30-second answer:** "The driver builds a DAG from my transformations; each action makes a job, cut into stages at shuffles, and each stage runs one task per partition on executors. I cache reused expensive DataFrames with `MEMORY_AND_DISK`, checkpoint to cut long lineage, and debug via the UI: stage skew, spill, GC and the SQL plan."

**Structure a longer answer:** architecture → job/stage/task → lazy evaluation → caching/checkpoint → memory/resources → debugging checklist → monitoring.

**Likely follow-ups:**
- *Cache vs checkpoint?* → performance vs. reliability and lineage truncation.
- *Container killed by YARN?* → off-heap/Python usage beyond overhead; raise `memoryOverhead`, reduce cores, fix skew/large broadcasts.
- *Why can `collect()` crash the driver?* → all rows land in driver heap; use aggregation or distributed write.
- *Client vs cluster mode?* → where the driver runs and logs land; production uses cluster.
- *Retry-safe pipeline?* → idempotent writes (overwrite partition / MERGE), no appended partial output.

**Signals interviewers look for:** correct vocabulary (job/stage/task), memory-model reasoning, evidence-based debugging.

### 11. Check yourself
- [ ] Given a script with 3 actions and 2 shuffles each, count jobs, stages and tasks for 100 partitions.
- [ ] Explain when cache helps and when it hurts.
- [ ] List the first four things you check in the UI for a job that slowed 10×.
- [ ] Explain why `memoryOverhead` matters for PySpark.
<!-- explainer:end -->

## Implementations (tools)

- [Apache Spark](<../_index.md>)
- [Hadoop](<../../Hadoop/_index.md>)

## Practice questions

- [Basic] [What does it mean that Spark is a unified engine and what workloads does it cover under one API?](<../../../PRACTICE/TOOLS/Apache_Spark/Spark_Core/Q-02444_What_does_it_mean_that_Spark_is_a_unified_engine_and_what_wo.md>)
- [Basic] [What is PySpark and how does it relate to the Spark core written in Scala?](<../../../PRACTICE/TOOLS/Apache_Spark/Spark_Core/Q-02301_What_is_PySpark_and_how_does_it_relate_to_the_Spark_core_wri.md>)
- [Intermediate] [What configuration properties control executor memory and cores in Spark?](<../../../PRACTICE/TOOLS/Apache_Spark/Spark_Core/Q-02304_What_configuration_properties_control_executor_memory_and_co.md>)
- [Intermediate] [How do you handle credential and secret management for a Spark job accessing multiple cloud data sources?](<../../../PRACTICE/TOOLS/Apache_Spark/Spark_Core/Q-02437_How_do_you_handle_credential_and_secret_management_for_a_Spa.md>)
- [Deep] [What is checkpointing in Spark and how does it differ from caching?](<../../../PRACTICE/TOOLS/Apache_Spark/Spark_Core/Q-02335_What_is_checkpointing_in_Spark_and_how_does_it_differ_from_c.md>)
- [Deep] [How do you decide between caching in memory, memory-and-disk, or not caching at all for a DataFrame reused three times?](<../../../PRACTICE/TOOLS/Apache_Spark/Spark_Core/Q-02374_How_do_you_decide_between_caching_in_memory_memory-and-disk_.md>)
- [Deep] [How would you implement data quality checks inside a Spark pipeline so bad batches fail fast rather than corrupt the warehouse?](<../../../PRACTICE/TOOLS/Apache_Spark/Spark_Core/Q-02401_How_would_you_implement_data_quality_checks_inside_a_Spark_p.md>)
- [Deep] [What metrics would you export from Spark to monitor job health over time and catch gradual regressions?](<../../../PRACTICE/TOOLS/Apache_Spark/Spark_Core/Q-02432_What_metrics_would_you_export_from_Spark_to_monitor_job_heal.md>)

## Related concepts

- [Spark Executor Sizing](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Spark_Executor_Sizing.md>) *(Big Data Processing)*
- [Spark DataFrames and Lazy Evaluation](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Spark_DataFrames_and_Lazy_Evaluation.md>) *(Pipeline Orchestration)*
- [Shuffle Partition Tuning](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Shuffle_Partition_Tuning.md>) *(Big Data Processing)*
- [Handling Poison Records](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Handling_Poison_Records.md>) *(Pipeline Orchestration)*
- [Profiling Spark Stages](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Profiling_Spark_Stages.md>) *(Pipeline Orchestration)*

## References

- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ingestion Design Patterns › Examples
- *Data Engineering Design Patterns* — Bartosz Konieczny — Idempotency Design Patterns › Solution
- *Data Engineering Design Patterns* — Bartosz Konieczny — Idempotency Design Patterns › Distributed processing
- *Data Engineering Design Patterns* — Bartosz Konieczny — Pattern: Stateful Sessionizer › Examples
- *Hadoop: The Definitive Guide* — Tom White — YARN › Building YARN Applications
- *Hadoop: The Definitive Guide* — Tom White — Spark › Chapter 19. Spark
- *Hadoop: The Definitive Guide* — Tom White — Spark › Spark Applications, Jobs, Stages, and Tasks
- *Hadoop: The Definitive Guide* — Tom White — Spark › A Scala Standalone Application
- *Data Engineering Fundamentals* — Zhaolong Liu — CHAPTER 4Data Transformation and Processing › CHAPTER 4Data Transformation and Processing(#contents.xhtmlch04a)
- *Data Engineering Fundamentals* — Zhaolong Liu — CHAPTER 8Monitoring and Performance › CHAPTER 8Monitoring and Performance(#contents.xhtmlch08a)
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Streaming In and Out of Your Delta Lake › Apache Spark
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Streaming In and Out of Your Delta Lake › Advanced Usage with Apache Spark
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Streaming In and Out of Your Delta Lake › Delta Lake Performance Metrics
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Streaming In and Out of Your Delta Lake › Custom metrics
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Batch Processing Patterns › 6. Batch Processing Patterns
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Introduction to Apache Spark A Unified Analytics Engine › Chapter 1. Introduction to Apache Spark: A Unified Analytics Engine
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Introduction to Apache Spark A Unified Analytics Engine › The Genesis of Spark
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Introduction to Apache Spark A Unified Analytics Engine › What Is Apache Spark?
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Introduction to Apache Spark A Unified Analytics Engine › Unified Analytics

