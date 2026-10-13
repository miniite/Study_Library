---
id: "TOOLS/Apache_Spark/01_Spark_Fundamentals/Spark_Architecture"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Spark Architecture and Execution

**L4 Advanced** · [↑ Spark Fundamentals](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A Spark application is one driver that turns your code into a DAG of stages and tasks, and many executors (started by a cluster manager) that run those tasks in parallel on partitions of the data.

### 1. The big picture
Spark's "Fundamentals" node explained *what* Spark is. This lesson explains *who does what* when a program runs. Everything you later tune (shuffles, memory, parallelism, skew) is a property of this machinery, so it is worth being able to draw it from memory.

Three layers matter. Your **code** builds a plan lazily. The **driver** compiles that plan into jobs, stages and tasks and keeps track of them. The **executors** (JVM processes on worker machines, handed out by a cluster manager such as YARN, Kubernetes or standalone) run the tasks and hold cached data. Compared with MapReduce, Spark's job is a general DAG rather than a fixed map→reduce pair, and the application stays alive across several jobs so cached data can be reused.

### 2. Core concepts and vocabulary
- **Application** — one user program (one `SparkContext`/`SparkSession`). It can run many jobs in series or in parallel and can reuse cached datasets between them. An interactive `spark-shell` session is just an application.
- **Driver** — the process that runs `main()`, owns the `SparkSession`, builds the plan, schedules tasks and collects results. If it dies, the application dies.
- **Executor** — a JVM on a worker node that runs tasks, stores cached partitions and shuffle files. Lives for the whole application (unless dynamic allocation removes it).
- **Cluster manager** — grants containers/pods/slots to the application: local, standalone, YARN, Kubernetes (Mesos in older releases).
- **Job** — the work triggered by one *action* (`count`, `write`, `collect`…).
- **Stage** — a set of tasks that can run without moving data between machines; stages are separated by *shuffles*.
- **Task** — the smallest unit of work: one stage's computation applied to **one partition**, running in one executor core.
- **Partition** — a slice of the dataset; the unit of parallelism and of caching.
- **Transformation / Action** — transformations are lazy and only extend the plan; actions force execution.
- **Narrow vs wide dependency** — narrow: each output partition depends on one input partition (`map`, `filter`); wide: it depends on many (`groupBy`, `join`), which needs a shuffle.
- **Deploy mode** — *client* (driver runs where you submitted from) or *cluster* (driver runs inside the cluster).

### 3. How it works, step by step
1. **Submit.** `spark-submit` (or a notebook) starts the driver. The driver asks the cluster manager for executors.
2. **Build the plan.** Each transformation adds a node to the logical plan / RDD lineage. Nothing runs yet.
3. **Action → job.** An action makes the driver hand the final dataset to the **DAG scheduler**.
4. **Cut into stages.** The scheduler walks backwards through the lineage and starts a new stage at every wide dependency. Stages form a DAG; a stage can only start when its parent stages are finished.
5. **Create tasks.** Each stage becomes a *task set*: one task per partition of the stage's output.
6. **Schedule tasks.** The **task scheduler** sends tasks to executors with free cores, preferring executors that hold the input block (data locality: process-local, node-local, rack-local, any).
7. **Execute.** The executor deserialises the task (the code plus any broadcast variables), reads its partition (from storage, cache, or shuffle files), computes, then either returns a result to the driver or writes shuffle output for the next stage.
8. **Recover.** If a task fails it is retried (default 4 attempts); if shuffle files are lost, the parent stage's missing partitions are recomputed from lineage.
9. **Finish.** When the last stage completes the action returns; the application keeps running and can run more jobs.

```
your code ──► driver (SparkSession, DAG scheduler, task scheduler)
                  │  asks for resources
                  ▼
           cluster manager ──► executor 1 [core][core]   cache + shuffle files
                              executor 2 [core][core]
                              executor N ...
```

![How Spark runs a job](<../../../../_assets/Hadoop_The_Definitive_Guide_Tom_White/hddg_1901.png>)

### 4. Worked example
```python
from pyspark.sql import SparkSession, functions as F

spark = SparkSession.builder.appName("arch-demo").getOrCreate()

df = spark.read.parquet("/data/flights")           # transformation-ish: only schema read
agg = (df.filter(F.col("delay") > 15)              # narrow
         .groupBy("origin")                        # wide -> shuffle
         .agg(F.count("*").alias("late_flights")))
agg.explain()        # look for Exchange hashpartitioning = the stage boundary
agg.write.mode("overwrite").parquet("/out/late")   # ACTION -> 1 job
```
What happens: the write is the action, so one **job** is created. `filter` and the partial aggregation are narrow and run in **stage 0**, with one task per input file split. The `Exchange` in the plan is the shuffle; the final aggregation and write run in **stage 1** with `spark.sql.shuffle.partitions` tasks (200 by default; with adaptive query execution Spark may coalesce them). In the Spark UI you see 1 job, 2 stages, and per-stage task counts equal to those partition numbers. Calling `agg.count()` afterwards launches a *second* job that recomputes the whole lineage unless the data was cached.

### 5. Under the hood
- **Lineage, not replication.** Fault tolerance comes from recomputation: each RDD remembers the transformations that produced it. This is cheap for narrow dependencies; for wide ones a lost partition may require re-running many parent tasks, which is why long lineages are sometimes checkpointed.
- **DataFrames sit on top.** A DataFrame query is optimised by Catalyst into a physical plan which is compiled to RDD operations (with whole-stage code generation fusing operators in a stage into one loop). The scheduler described above is the same.
- **Executor memory** is split into execution (shuffles, joins, sorts), storage (cache) and user/reserved space; execution and storage borrow from each other under the unified memory manager.
- **Shuffle** writes map-side output to local disk files and reduce-side tasks fetch them over the network. The external shuffle service (or YARN/K8s equivalents) lets files outlive executors, which is required for dynamic allocation.
- **Speculation** (`spark.speculation`) can launch a duplicate of a slow task; useful with stragglers, risky with non-idempotent side effects.
- **Dynamic allocation** adds/removes executors according to backlog of pending tasks.

### 6. Performance and design implications
- Parallelism = number of partitions × cores available. Too few partitions leaves cores idle; too many creates scheduling overhead. A common target is 2–4 tasks per core and partitions of roughly 100–200 MB.
- The number of **stages** is driven by shuffles: fewer shuffles → fewer stage boundaries → less disk and network I/O.
- Driver bottlenecks: `collect()` of large results, huge numbers of tiny tasks, or giant broadcast variables overload one JVM.
- Locality matters on HDFS-style clusters; on object stores (S3/ADLS) there is no locality and throughput comes from parallel reads.
- Measure with the Spark UI: Jobs → Stages → task timeline, shuffle read/write sizes, GC time, and per-task duration distribution (skew shows as one very long task).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `spark-submit --master yarn --deploy-mode cluster` | Run the driver inside the cluster | Production jobs, so the driver is not tied to your laptop |
| `--num-executors`, `--executor-cores`, `--executor-memory` | Size the executor fleet | Static allocation tuning |
| `spark.dynamicAllocation.enabled` | Scale executors with load | Shared clusters, variable workloads |
| `spark.sql.shuffle.partitions` | Tasks after a shuffle (default 200) | Adjust to data volume; AQE can coalesce |
| `spark.default.parallelism` | Default partitions for RDD shuffles | RDD-based jobs |
| `df.explain()` / `explain("formatted")` | Show plans and Exchange nodes | Find stage boundaries |
| `df.rdd.getNumPartitions()` | Inspect partition count | Before/after `repartition` |
| `spark.task.maxFailures` | Task retry limit | Flaky infrastructure |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Cluster deploy mode for scheduled production jobs | Client mode over a slow network: driver↔executor traffic suffers |
| Few big executors (more cores, shared memory) | Very large heaps (>~32 GB) cause long GC pauses |
| Many small executors | Per-executor overhead and replicated broadcast data grow |
| Dynamic allocation on shared clusters | Needs shuffle service; startup latency for streaming/latency-sensitive jobs |
| Speculation for stragglers on read-heavy jobs | Writes to non-idempotent sinks |

### 9. Common mistakes
- Treating transformations as if they executed → timing the wrong line → time the action, or look at the UI.
- `collect()` on a big dataset → driver OOM → write out, or aggregate first, or `take(n)`.
- Calling several actions on one uncached lineage → the pipeline recomputes each time → cache/persist the reused dataset (see the caching lesson).
- Leaving the default 200 shuffle partitions on a tiny or enormous job → tiny tasks or spills → size to the data (or rely on AQE).
- Assuming one task = one row or one file → misreading UI → one task = one partition.

### 10. 🎯 Interview kit
**30-second answer:** A Spark app has a driver and executors. The driver builds a lazy DAG; an action triggers a job, which is split into stages at shuffle boundaries, and each stage into one task per partition. Tasks are scheduled on executors provided by a cluster manager, run in parallel, and are recomputed from lineage if they fail.
**Structure a longer answer:** clarify (batch job? which cluster manager?) → driver / executor / cluster-manager roles → job→stage→task decomposition and why shuffles split stages → fault tolerance by lineage and retries → tuning levers (partitions, executor sizing) → a real scenario such as a skewed join stage with one slow task.
**Likely follow-ups:**
- *Narrow vs wide dependency?* → one-to-one partition mapping vs shuffle across partitions; wide makes a stage boundary.
- *What happens when an executor dies?* → its tasks are rescheduled; lost shuffle/cached data is recomputed from lineage (or fetched from the external shuffle service).
- *Client vs cluster mode?* → where the driver runs; cluster for production, client for interactive.
- *What is the unit of parallelism?* → the partition; one task per partition per stage.
- *Why is the application long-lived beyond a job?* → so cached data and executors are reused across jobs.
**Signals interviewers look for:** you separate logical plan from execution, can name what causes a stage boundary, connect symptoms in the UI (straggler task, spill, GC) to causes, and know the driver is a single point of failure and bottleneck.

### 11. Check yourself
- [ ] Draw driver, cluster manager and executors and label where the DAG scheduler and task scheduler live.
- [ ] Given a pipeline `read → filter → groupBy → join → write`, count the stages and say why.
- [ ] Explain why `df.count()` twice on an uncached DataFrame does the work twice.
- [ ] Say what is lost and what is recomputed when one executor is killed mid-stage.
- [ ] Find the `Exchange` node in an `explain()` and map it to a stage boundary in the UI.
<!-- explainer:end -->

## In this section (study in order)

1. [Jobs Stages and Tasks](<Jobs_Stages_Tasks/_index.md>) (L5 Expert)

## References

- *Hadoop: The Definitive Guide* — Tom White — Spark › Anatomy of a Spark Job Run
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Introduction to Apache Spark A Unified Analytics Engine › Spark driver
- *Hadoop: The Definitive Guide* — Tom White — Spark › Executors and Cluster Managers
- *Hadoop: The Definitive Guide* — Tom White — Spark › Note
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Downloading Apache Spark and Getting Started › Spark Application and SparkSession
- *Hadoop: The Definitive Guide* — Tom White — Spark › Spark Applications, Jobs, Stages, and Tasks
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Downloading Apache Spark and Getting Started › Spark Tasks
- *Hadoop: The Definitive Guide* — Tom White — Spark › Task Scheduling
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Optimizing and Tuning Spark Applications › Scaling Spark for Large Workloads
- *Hadoop: The Definitive Guide* — Tom White — Spark › Transformations and Actions

