---
id: "TOOLS/Apache_Spark/02_Spark_Tuning/Resources_and_Parallelism"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Resources Memory and Parallelism

**L4 Advanced** · [↑ Spark Tuning](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Spark's speed comes from running one task per partition per core, so tuning resources means sizing executors, deciding static vs dynamic allocation, and making sure there are enough (but not too many) well-sized partitions to keep every core busy.

### 1. The big picture
A Spark job is fast when all cores are busy with similarly sized units of work and slow when cores idle, tasks are skewed, or memory pressure causes spills and GC. The three levers are: **how many resources you ask for** (executors, cores, memory), **how those resources are acquired** (fixed vs dynamic), and **how data is cut into partitions** (read partitions, shuffle partitions, `repartition`/`coalesce`). This is the same principle that makes any scale-out system fast: parallel execution of independent units with a bottleneck at the slowest unit.

### 2. Core concepts and vocabulary
- **Core / slot** — one task thread; an executor with 4 cores runs 4 tasks at a time.
- **Partition** — an atomic unit of parallelism: one core works on one partition at a time.
- **Task wave** — one round of tasks filling all cores; 1,000 tasks on 100 cores = 10 waves.
- **Static allocation** — a fixed number of executors for the whole application.
- **Dynamic allocation** — executors added when tasks queue and removed when idle.
- **Shuffle partitions** — number of partitions after a wide transformation (`spark.sql.shuffle.partitions`).
- **Max partition bytes** — target size for file-read partitions (`spark.sql.files.maxPartitionBytes`, default 128 MB).
- **Skew** — uneven partition sizes, giving a few very slow tasks.
- **Executor overhead / off-heap** — memory outside the JVM heap for the container (`spark.executor.memoryOverhead`).

### 3. How it works, step by step
1. You ask the cluster manager for N executors × C cores × M memory (via `spark-submit` flags or configs).
2. The driver counts available task slots = executors × cores.
3. Each stage creates one task per partition. Input partitions come from file splits (about 128 MB each); post-shuffle partitions come from `spark.sql.shuffle.partitions` (or AQE-coalesced).
4. Tasks are dispatched to free slots; the stage finishes when the slowest task finishes.
5. With dynamic allocation: if tasks are pending longer than `schedulerBacklogTimeout`, the driver requests more executors (exponentially, up to `maxExecutors`); executors idle longer than `executorIdleTimeout` are released (down to `minExecutors`).
6. Ideal state: partitions ≥ total cores (the book's rule: at least as many partitions as cores) and each partition small enough to fit execution memory without spilling.

### 4. Worked example
```bash
# Static: fixed fleet
spark-submit --master yarn --deploy-mode cluster \
  --num-executors 10 --executor-cores 4 --executor-memory 8g \
  --conf spark.executor.memoryOverhead=1g job.py
# slots = 10 x 4 = 40 concurrent tasks

# Dynamic: scale with the backlog (shuffle service or K8s shuffle tracking required)
spark-submit ... \
  --conf spark.dynamicAllocation.enabled=true \
  --conf spark.dynamicAllocation.minExecutors=2 \
  --conf spark.dynamicAllocation.maxExecutors=20 \
  --conf spark.dynamicAllocation.schedulerBacklogTimeout=1m \
  --conf spark.dynamicAllocation.executorIdleTimeout=2min \
  --conf spark.dynamicAllocation.shuffleTracking.enabled=true
```
```python
print(df.rdd.getNumPartitions())          # e.g. 6 on a 40-slot cluster -> 34 cores idle
df = df.repartition(160)                  # full shuffle, even sizes (4 waves of 40)
small = df.coalesce(8)                    # narrow, no shuffle, only reduces
spark.conf.set("spark.sql.shuffle.partitions", "400")   # post-shuffle task count
```
Interpretation: a 6-partition read on 40 slots uses 15% of the cluster; repartitioning to a multiple of the slot count balances waves. After a shuffle on 2 TB, 200 partitions of 10 GB each will spill; 16,000 partitions of 128 MB is healthy. In the Spark UI, check the stage's task count, the timeline (gaps = idle cores), and spill columns.

### 5. Under the hood
- **Partition arithmetic for file reads:** Spark computes split size ≈ `max(openCostInBytes, min(maxPartitionBytes, totalBytes / parallelism))`, packs small files together and splits large splittable files (Parquet, ORC, plain text; not gzip). Non-splittable compressed files give one task per file.
- **Shuffle partitions** are chosen before the data size is known; AQE coalesces small post-shuffle partitions using runtime statistics (`spark.sql.adaptive.coalescePartitions.enabled`) and can split skewed ones in joins.
- **Executor sizing:** each core shares the executor's memory, so memory per task ≈ (execution memory share) / cores. Large heaps lengthen GC; very small executors multiply per-JVM overhead and broadcast copies. Common practice is 3–5 cores per executor and heaps in the 8–32 GB range, leaving room for OS/overhead.
- **Dynamic allocation** needs the external shuffle service (or shuffle tracking/K8s support) so shuffle files outlive the executor that wrote them. Executors with cached data are normally exempt (`cachedExecutorIdleTimeout`).
- **Parallelism beyond Spark:** reading from a database or object store is limited by source concurrency; more partitions can hurt a JDBC source or hit S3 request limits.
- Spark is "embarrassingly parallel" only per stage; Amdahl's law applies to the driver-side and shuffle phases.

### 6. Performance and design implications
- Target roughly 2–4 tasks per core and 100–200 MB per partition; scale `shuffle.partitions` with the shuffled volume.
- Under-partitioned: idle cores, OOM, large spills. Over-partitioned: scheduler overhead, tiny files on write, excess shuffle fetches.
- Use `coalesce` to reduce partitions before writing (no shuffle); use `repartition` (with a column) to rebalance or colocate keys.
- Dynamic allocation improves cluster utilisation for bursty or multi-tenant workloads, but startup latency and executor churn hurt latency-sensitive streaming; consider fixed fleets there.
- Watch per-task metrics: duration spread (skew), GC time (>10% suggests heap pressure), shuffle spill (memory too small per task), input size per task.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `--num-executors`, `--executor-cores`, `--executor-memory` | Size static fleet | Predictable batch jobs |
| `spark.dynamicAllocation.enabled`, `min/maxExecutors` | Elastic scaling | Variable load, shared clusters |
| `spark.dynamicAllocation.schedulerBacklogTimeout`, `executorIdleTimeout` | Scale-up / scale-down timers | Tune responsiveness |
| `spark.sql.shuffle.partitions` | Partitions after shuffle | Match to shuffled data volume |
| `spark.sql.files.maxPartitionBytes` | Input split size | Fewer/more read tasks |
| `spark.sql.adaptive.coalescePartitions.enabled` | AQE merges small partitions | Default on in 3.x |
| `df.repartition(n[, cols])` / `df.coalesce(n)` | Change partition count | Rebalance / reduce |
| `spark.executor.memoryOverhead` | Non-heap memory | Container kills (YARN/K8s OOM) |
| `spark.default.parallelism` | RDD default partitions | RDD workloads |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Static allocation: steady, scheduled jobs, streaming | Idle executors waste paid capacity on bursty loads |
| Dynamic allocation: multi-tenant, spiky analytics | Needs shuffle service/tracking; resource churn can affect neighbours |
| `repartition(n)` to fix skew/few partitions | A full shuffle is expensive; avoid just before a write if you only need fewer files (use `coalesce`) |
| More, smaller partitions to avoid spill | Too many tiny tasks add overhead |
| Fewer fat executors for broadcast-heavy jobs | Long GC pauses with huge heaps |

### 9. Common mistakes
- Leaving 200 shuffle partitions on multi-TB jobs → giant partitions that spill → raise it or rely on AQE.
- `coalesce(1)` before writing a large dataset → one core does all the work → write many files and compact later.
- Counting cores but ignoring memory per core → OOM → fewer cores per executor or more memory.
- Reading many tiny files → huge task count and listing time → compact files or raise `openCostInBytes`/use bin-packing.
- Believing more executors always means faster → source or skew becomes the bottleneck.
- Setting dynamic allocation with no shuffle service → shuffle fetch failures after scale-down.

### 10. 🎯 Interview kit
**30-second answer:** Spark runs one task per partition per core, so throughput depends on having at least as many evenly sized partitions as cores and enough memory per task. You size executors (cores/memory), choose static or dynamic allocation, and tune input and shuffle partitions, using the UI to spot idle cores, skew and spills.
**Structure a longer answer:** clarify workload (batch/stream, data size, shared cluster) → partitions as units of parallelism → executor sizing math → static vs dynamic trade-offs → partition tuning tools (`repartition`, `coalesce`, AQE) → how you verify (UI metrics).
**Likely follow-ups:**
- *repartition vs coalesce?* → full shuffle and rebalance vs narrow merge that only reduces.
- *How do you pick shuffle partitions?* → shuffled bytes / 100–200 MB, at least a few × cores, or AQE.
- *Why not one giant executor?* → GC and HDFS client throughput, single failure domain.
- *What does dynamic allocation need?* → shuffle service/tracking.
- *Why would adding executors not speed up a job?* → too few partitions, skew, source bottleneck, driver bound.
**Signals interviewers look for:** concrete numbers (cores, partition sizes), awareness of memory per task, and verifying with metrics rather than guessing.

### 11. Check yourself
- [ ] Compute task slots and number of waves for 1,000 tasks on 12 executors × 4 cores.
- [ ] Explain the dynamic allocation settings and what each timer does.
- [ ] Choose a shuffle partition count for 500 GB of shuffled data.
- [ ] Decide between `repartition` and `coalesce` for writing 50 output files from 2,000 partitions.
- [ ] Identify in the UI a stage that is under-partitioned.
<!-- explainer:end -->

## References

- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Optimizing and Tuning Spark Applications › Static versus dynamic resource allocation
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Optimizing and Tuning Spark Applications › Maximizing Spark parallelism
- *Designing Data-Intensive Applications* — Martin Kleppmann — Graphs and Iterative Processing › Parallel execution
- *Designing Data-Intensive Applications* — Martin Kleppmann — Encoding and Evolution › Scaling to Higher Load

