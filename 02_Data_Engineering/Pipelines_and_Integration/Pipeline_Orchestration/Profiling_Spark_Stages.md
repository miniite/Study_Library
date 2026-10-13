---
id: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Profiling_Spark_Stages"
level: 4
type: "concept"
topic: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration"
tools: ["Apache_Spark", "Apache_Airflow", "Kubernetes", "Python"]
---

# Profiling Spark Stages

**L4 Advanced** · [↑ Pipeline Orchestration](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** To tune a slow Spark job, find the slowest stage in the Spark UI and read its task metrics to see whether time goes to input, shuffle, skew, spill, garbage collection, or serialization.

### 1. The big picture
Spark tuning is diagnosis first, configuration second. A job is split into **stages** at shuffle boundaries; each stage runs many **tasks** in parallel, one per partition. "The job is slow" is almost always "one stage is slow", and within that stage either every task is slow (too little resource, expensive work per row) or a few tasks are far slower than the rest (skew, stragglers). Profiling means reading metrics at the right level — job → stage → task → executor — and matching the pattern to a cause.

This lesson connects the resource knobs (executors, cores, memory, dynamic allocation, shuffle service) to the symptoms you will see, so that a change is a response to evidence rather than folklore.

### 2. Core concepts and vocabulary
- **Job / stage / task** — an action creates a job; shuffles split it into stages; each partition in a stage is one task.
- **Shuffle** — redistribution of data across executors by key (joins, groupBy, repartition); written to local disk then fetched over the network.
- **Shuffle read/write size** — bytes moved between stages; large numbers signal expensive wide operations.
- **Spill** — when execution memory is insufficient, Spark writes intermediate data to disk (memory spill vs disk spill metrics).
- **GC time** — time the JVM spent in garbage collection inside tasks; high values mean memory pressure.
- **Skew** — uneven partition sizes; the max task duration far exceeds the median.
- **Straggler** — a single task holding up a stage.
- **Executor** — JVM process with N cores and a memory budget that runs tasks.
- **Execution vs storage memory** — regions for shuffles/joins/sorts/aggregations vs cached data; they borrow from each other.
- **Dynamic allocation** — Spark adds/removes executors based on pending tasks.
- **External shuffle service** — keeps shuffle files available when executors are removed.
- **AQE (adaptive query execution)** — runtime re-optimization (coalescing partitions, skew-join handling, join strategy switch).

### 3. How it works, step by step
A reliable profiling routine:
1. **Identify the slow stage.** Spark UI → Jobs → click the long job → Stages sorted by duration.
2. **Open the stage summary metrics.** Compare Min / 25th / Median / 75th / Max for: duration, input size, shuffle read, spill, GC time.
3. **Classify the pattern.**
   - Max ≫ median in duration and size → **skew**.
   - All tasks similar but slow, high shuffle read/write → **shuffle-bound**.
   - High "Spill (disk)" → **insufficient execution memory** or too-large partitions.
   - GC time > ~10% of task time → **memory pressure / too many objects**.
   - Low CPU, long "scheduler delay" or "getting result" → **too many tiny tasks** or driver overhead.
   - Python/UDF heavy stages with high serialization time → **serialization-bound**.
4. **Read the SQL tab / plan** (`explain("formatted")`) to see which operator created the shuffle (SortMergeJoin, HashAggregate, Exchange).
5. **Change one thing**, rerun, and compare stage metrics.
6. **Scale only when per-task work is already efficient** — adding executors doesn't fix skew.

### 4. Worked example
```python
spark.conf.set("spark.sql.adaptive.enabled", "true")
spark.conf.set("spark.sql.shuffle.partitions", "400")

features = (events
    .join(services, "service_id")            # wide: shuffle
    .groupBy("service_id", "hour")           # wide: shuffle
    .agg(F.count("*").alias("n"), F.avg("latency").alias("p")))

features.write.mode("overwrite").parquet("s3://lake/features/")
```
After running, you open the Stages tab and see stage 14 (the groupBy) taking 22 minutes. Summary metrics:

| Metric | Median | Max |
|---|---|---|
| Duration | 18 s | 21 min |
| Shuffle read | 120 MB | 6.8 GB |
| Spill (disk) | 0 | 4.1 GB |
| GC time | 0.8 s | 140 s |

Max ≫ median in size and time, with spill and GC concentrated in the heavy task: **one hot key** (a service that emits most events). Remedies in order: let AQE's skew-join handling split it; salt the key (`service_id` + random 0..N) and aggregate twice; or pre-aggregate before the join. Adding executors wouldn't have helped, because one task still had to process 6.8 GB.

### 5. Under the hood
- **Executor memory layout:** `spark.executor.memory` is the JVM heap. About 300 MB is reserved. Of the rest, `spark.memory.fraction` (default 0.6) forms the *unified region* shared by execution and storage; `spark.memory.storageFraction` (default 0.5) sets the soft boundary, and either side can borrow when the other is idle (execution can evict cached blocks down to that boundary; storage cannot evict execution). The remaining ~40% is user memory for data structures and UDF objects. Simplified descriptions in books state this as a 60/40 split of memory; the unified-region behavior is what matters when diagnosing spills.
- **Shuffle mechanics:** map tasks write sorted, partitioned files on local disk; reduce tasks fetch their partition from every map output. Cost grows with map-count × reduce-count blocks, which is why thousands of tiny partitions are slow.
- **Spill:** sort/aggregate buffers exceeding execution memory are written to disk and merged later, trading speed for survival.
- **Dynamic allocation** needs the external shuffle service (or shuffle tracking) so removed executors don't lose shuffle files.
- **Python UDFs** move rows between the JVM and Python workers via serialization; built-in functions and pandas UDFs (Arrow) avoid most of that.

### 6. Performance and design implications
- **Partition size target:** 100–200 MB per partition after shuffle; default `spark.sql.shuffle.partitions = 200` is often wrong for both small and huge jobs (AQE coalescing helps).
- **Executor sizing:** moderate executors (e.g., 4–5 cores, 8–32 GB) usually balance parallelism, GC pause, and HDFS/S3 throughput better than one giant JVM per node.
- **Static allocation** for critical, predictable production jobs; **dynamic allocation** with min/max bounds for variable analytic or ad-hoc work.
- **Cost vs runtime:** more executors reduce wall time with diminishing returns once shuffle or skew dominates; spot/preemptible nodes cut cost but need shuffle-safe design.
- **Layout for serving:** write partitioning is a separate decision from compute partitioning; re-partition output for the read pattern (e.g., by lookup key, with compacted files).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| Spark UI → Stages / SQL / Executors | Per-stage and per-task metrics | First step in any diagnosis |
| `df.explain("formatted")` | Shows physical plan and Exchanges | Locate shuffles and join types |
| `spark.sql.shuffle.partitions` | Post-shuffle partition count | Tune for data volume |
| `spark.sql.adaptive.enabled` (+ skew join, coalesce) | Runtime re-optimization | Default on in modern Spark |
| `spark.executor.memory`, `.cores`, `.memoryOverhead` | Executor resources | Spill/OOM/GC fixes |
| `spark.dynamicAllocation.*` | Elastic executors | Variable workloads |
| `spark.memory.fraction`, `storageFraction` | Memory regions | Cache-heavy or shuffle-heavy jobs |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| More partitions — tasks spill or are too large | Tasks become tiny; scheduler overhead rises |
| Bigger executors — heavy per-task memory need | GC pauses grow with heap size |
| Salting — one hot key | Keys are evenly distributed; you only add work |
| Broadcast join — one side small | "Small" side actually grows; driver/executor OOM |
| Cache — reused DataFrame | Single use; it steals execution memory |

### 9. Common mistakes
- Adding memory without looking at the stage → same straggler, higher bill → read the max/median first.
- Leaving 200 shuffle partitions on a 5 TB shuffle → huge tasks, spills → raise partitions or rely on AQE.
- Over-caching → eviction and GC → cache only reused, expensive results and unpersist.
- Python UDF for logic available in built-ins → serialization-bound stage → use functions or pandas UDF.
- Blaming Spark for a slow source → input stage waits on I/O → check input read time and file counts (small files).

### 10. 🎯 Interview kit
**30-second answer:** Open the Spark UI, find the slowest stage, and compare min/median/max task metrics. A big gap means skew; large shuffle read/write means shuffle-bound; disk spill means memory/partition-size problems; high GC time means memory pressure; high serialization time points at UDFs. Fix the specific cause, then re-measure.

**Structure a longer answer:** clarify (batch/stream, data size, what changed) → core mechanism (jobs/stages/tasks, shuffles) → trade-offs (executor sizing, partitions, AQE) → real scenario (3× slower after a volume spike).

**Likely follow-ups:**
- *Skew vs GC pressure?* → skew: uneven task durations/sizes; GC: high GC time across or within tasks, and spills.
- *How do you pick executors/cores/memory?* → total cores vs data size, ~100–200 MB partitions, 4–5 cores per executor, test and measure cost × runtime.
- *Static vs dynamic allocation?* → predictable critical jobs vs variable workloads.
- *Straggler with a single task?* → skew, a bad node, or an expensive partition; speculation can help the node case.

**Signals interviewers look for:** evidence-driven, knows where the metrics live, avoids "just add nodes".

### 11. Check yourself
- [ ] Describe the steps from slow job to root cause using the Spark UI.
- [ ] Explain execution vs storage memory and how they borrow.
- [ ] List three fixes for skew and when each applies.
- [ ] Explain why dynamic allocation needs a shuffle service.
<!-- explainer:end -->

## Implementations (tools)

- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)
- [Apache Airflow](<../../../TOOLS/Apache_Airflow/_index.md>)
- [Kubernetes](<../../../TOOLS/Kubernetes/_index.md>)
- [Python](<../../../TOOLS/Python/_index.md>)

## Practice questions

- [Basic] [What is a cluster manager in Spark (like YARN or Kubernetes) at a beginner level?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13392_What_is_a_cluster_manager_in_Spark_like_YARN_or_Kubernetes_a.md>)
- [Deep] [How do you choose the windowing strategy for a Spark job that groups related alerts by time proximity so flapping services don't create runaway clusters?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13446_How_do_you_choose_the_windowing_strategy_for_a_Spark_job_tha.md>)
- [Deep] [Your Spark job spills heavily to disk while computing per-host metric percentiles; what levers do you pull first before adding memory?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13482_Your_Spark_job_spills_heavily_to_disk_while_computing_per-ho.md>)
- [Deep] [How do you handle a Spark job that reads metrics from a rate-limited monitoring API that can't be parallelized without overloading the source?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13507_How_do_you_handle_a_Spark_job_that_reads_metrics_from_a_rate.md>)
- [Deep] [How would you profile a Spark stage in the log-clustering job to decide whether the bottleneck is I/O shuffle or serialization?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13530_How_would_you_profile_a_Spark_stage_in_the_log-clustering_jo.md>)
- [Deep] [How do you decide between running Spark on EMR self-managed clusters versus EMR Serverless or Databricks jobs compute for a bursty daily batch workload?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13672_How_do_you_decide_between_running_Spark_on_EMR_self-managed_.md>)
- [Deep] [How would you diagnose whether a slow Spark stage is bottlenecked on shuffle read GC pause or executor CPU using only the Spark UI and event logs?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13695_How_would_you_diagnose_whether_a_slow_Spark_stage_is_bottlen.md>)
- [Deep] [How do you decide Spark executor count cores per executor and memory for a feature job to balance cost against runtime?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13148_How_do_you_decide_Spark_executor_count_cores_per_executor_an.md>)

## Related concepts

- [Spark Executor Sizing](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Spark_Executor_Sizing.md>) *(Big Data Processing)*
- [Spark Execution Caching and Debugging](<../../../TOOLS/Apache_Spark/Spark_Core/Spark_Execution_Caching_and_Debugging.md>) *(Spark Core)*
- [Spark Skew and Spill](<../../../TOOLS/Apache_Spark/Spark_Optimization/Spark_Skew_and_Spill.md>) *(Spark Optimization)*

## References

- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Optimizing and Tuning Spark Applications › Scaling Spark for Large Workloads
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Optimizing and Tuning Spark Applications › Static versus dynamic resource allocation
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Optimizing and Tuning Spark Applications › Configuring Spark executors' memory and the shuffle service
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Optimizing and Tuning Spark Applications › Executors
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Infrastructure Operations › Understanding resource contention and allocation strategies
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — DataOps and AI › Scaling with AI-driven resource management at ACME
- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Learning about Machine Learning and Graph Processing in Databricks › Section 2: Optimization Techniques
- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Managing Spark Clusters › Designing Spark clusters
- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Managing Spark Clusters › Designing a Spark cluster for data analytics
- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Managing Spark Clusters › Designing a Spark cluster for a complex batch ETL process
- *Data Algorithms with Spark* — Mahmoud Parsian — **PART I** › PART I
- *Data Algorithms with Spark* — Mahmoud Parsian — **Serverless Spark** › Serverless Spark
- *Data Algorithms with Spark* — Mahmoud Parsian — **PART II** › PART II
- *Hadoop: The Definitive Guide* — Tom White — Setting Up a Hadoop Cluster › User Jobs
- *Hadoop: The Definitive Guide* — Tom White — Crunch › Running a Pipeline
- *Hadoop: The Definitive Guide* — Tom White — Spark › Task Scheduling
- *Databricks Certified Associate Developer for Apache Spark* — Saba Shah — Spark components › Spark executors
- *Databricks Certified Associate Developer for Apache Spark* — Saba Shah — Explain the optimized query plan › Memory management and tuning

