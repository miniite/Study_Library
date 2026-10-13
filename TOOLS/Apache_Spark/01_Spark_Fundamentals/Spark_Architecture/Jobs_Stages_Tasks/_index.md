---
id: "TOOLS/Apache_Spark/01_Spark_Fundamentals/Spark_Architecture/Jobs_Stages_Tasks"
level: 5
type: "curriculum"
curriculum: "data_engineering"
---

# Jobs Stages and Tasks

**L5 Expert** · [↑ Spark Architecture and Execution](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** An *action* triggers a **job**; Spark cuts the job's DAG into **stages** at shuffle boundaries; each stage runs one **task** per partition on the executors.

### 1. The big picture
Transformations (`select`, `filter`, `groupBy`) are lazy: they only extend a plan. Nothing executes until an **action** (`count`, `collect`, `write`, `show`) is called. At that moment the driver turns the plan into a physical execution that follows a strict hierarchy:

**application → job → stage → task**

Knowing this hierarchy is the key to reading the Spark UI: "why did my one line of code create 3 jobs, 7 stages and 2,000 tasks, and which one is slow?"

### 2. Core concepts and vocabulary
- **Application** — one SparkContext/SparkSession; has one driver and N executors.
- **Job** — the set of work launched by *one action*.
- **Stage** — a group of tasks that can run without moving data between executors; separated from the next stage by a **shuffle**.
- **Task** — the smallest unit of work: one stage applied to one partition; runs on one core of an executor.
- **Narrow dependency** — each output partition depends on one input partition (`map`, `filter`, `select`); pipelined inside a stage.
- **Wide dependency** — an output partition depends on many input partitions (`groupBy`, `join`, `repartition`); requires a shuffle.
- **Shuffle map task** — writes its output partitioned by key to local disk for the next stage.
- **Result task** — runs in the final stage and returns/writes the result.
- **DAG scheduler / task scheduler** — build stages from lineage / assign tasks to executor slots.
- **Locality level** — PROCESS_LOCAL, NODE_LOCAL, RACK_LOCAL, ANY.

### 3. How it works, step by step
1. You call an action. The driver takes the optimized plan (for DataFrames: physical plan → RDD graph).
2. The **DAG scheduler** walks the lineage backwards; every wide dependency becomes a **stage boundary**.
3. Stages are submitted in dependency order (parent stages first).
4. For each stage the **task scheduler** creates one task per partition and offers them to executors that have free cores, preferring data-local placement.
5. Shuffle-map tasks write shuffle files; the next stage's tasks fetch the blocks they need over the network.
6. The last stage's result tasks send results to the driver or write to storage.
7. Failed tasks are retried (default 4 times); lost shuffle files cause the parent stage to be recomputed.

Number of tasks in a stage = number of partitions of that stage's RDD/DataFrame. After a shuffle that is `spark.sql.shuffle.partitions` (default 200) unless AQE coalesces them.

### 4. Worked example
```python
df = spark.read.parquet("/data/events")              # lazy
clicks = df.filter("type = 'click'")                 # narrow
by_user = clicks.groupBy("user_id").count()          # wide  -> shuffle
top = by_user.orderBy("count", ascending=False)      # wide  -> another shuffle
top.limit(10).show()                                 # ACTION -> job starts
```
What happens:
- **1 job** (one action). `show` may launch extra small jobs when `limit` needs more partitions.
- **Stages**: (1) read + filter + partial aggregation, (2) final aggregation after shuffle + sort-sampling, (3) sort/limit — roughly three stages, boundaries at `groupBy` and `orderBy`.
- **Tasks**: stage 1 has one per input file split (e.g., 64); stage 2 has 200 shuffle partitions (or fewer with AQE).

Check it:
```python
top.explain("formatted")        # look for "Exchange" nodes = shuffles = stage boundaries
```
In the Spark UI → *Jobs* tab → click a job → DAG visualisation: each box is a stage; the *Stages* tab shows task counts, input/shuffle read/write, duration quantiles.

### 5. Under the hood   *(L5)*
- **Pipelining:** narrow operators in a stage are fused into one function per partition (with whole-stage codegen for DataFrames) so a record flows through filter→project→partial-agg without materialisation.
- **ShuffleMapStage vs ResultStage:** shuffle map stages produce `MapStatus` objects (sizes per reducer) that the driver tracks in the `MapOutputTracker`; reducers ask it where to fetch.
- **Stage reuse:** if a shuffle's output files still exist (e.g., you re-run an action on the same cached lineage), Spark marks the stage *skipped* in the UI — shuffle files act as an implicit cache.
- **Speculation:** with `spark.speculation=true`, slow tasks are re-launched on other executors; first finisher wins.
- **Barrier mode:** ML stages that need all tasks together (`barrier()`), differ from the default independent-task model.
- **Dynamic allocation:** executors are added when tasks are pending and removed when idle; shuffle files then rely on the external shuffle service or shuffle tracking.
- **Task serialization:** closures are serialised to executors; large captured objects slow launch (use broadcast variables).

### 6. Performance and design implications
- **Task count vs cores:** aim for 2–4 tasks per core; too few → idle cores, too many → scheduling overhead; target ~100–200 MB per partition.
- **Skew shows as straggler tasks:** max task time ≫ median in the Stages tab.
- **Each shuffle = disk + network:** minimise wide transformations; filter and project before them.
- **Many small jobs:** one action per iteration of a driver loop adds overhead; batch work.
- **Read these UI metrics:** *Duration* quantiles, *GC time*, *Shuffle Read/Write size*, *Spill (memory/disk)*, *Locality level*.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `df.explain("formatted")` | Show plan and `Exchange`s | Count stages before running |
| Spark UI → Jobs/Stages/SQL | Observe jobs, stages, tasks | Diagnosing slowness |
| `spark.sql.shuffle.partitions` | Post-shuffle task count | Tune parallelism |
| `spark.sql.adaptive.enabled` | AQE coalescing/skew handling | Default on in 3.x |
| `df.rdd.getNumPartitions()` | Partition count | Check input parallelism |
| `spark.task.maxFailures` | Retry limit | Flaky infra |
| `spark.speculation` | Duplicate slow tasks | Heterogeneous clusters |
| `repartition(n)` / `coalesce(n)` | Change partitions | Parallelism / small files |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| More shuffle partitions | Large data/skew; each task would be too big |
| Fewer shuffle partitions | Small data; scheduling overhead dominates |
| `coalesce` | Reducing partitions without a full shuffle |
| `repartition` | Need even sizes or key co-location (costs a shuffle) |
| Speculation | Stragglers from hardware; avoid when tasks are non-idempotent writes |

### 9. Common mistakes
- **Confusing transformations with actions** → "nothing happens" or work repeated per action → cache reused results.
- **Calling `collect()` on big data** → driver OOM → write out or aggregate first.
- **Leaving 200 shuffle partitions for terabytes** → huge tasks, spill → raise partitions / enable AQE.
- **Ignoring skewed stage tails** → one task runs for an hour → salting, AQE skew join.
- **Counting tasks wrongly** → expecting tasks per row/executor → it's per *partition*.

### 10. 🎯 Interview kit
**30-second answer:** "An action creates a job; the DAG scheduler splits it into stages at shuffle boundaries; each stage runs one task per partition on executor cores. Narrow ops pipeline inside a stage; wide ops force a shuffle and a new stage."

**Structure a longer answer:** define the hierarchy → how stage boundaries are found → how task counts arise → how to read it in the UI → tuning implications (partitions, skew).

**Likely follow-ups:**
- *How many stages does a groupBy + join + orderBy have?* → Count `Exchange` operators + 1 (broadcast joins add none).
- *Why was a stage skipped?* → Its shuffle output was reused.
- *What decides the number of tasks?* → Partition count of the stage's input.
- *Job vs stage vs task?* → Action / shuffle-bounded group / per-partition unit.

**Signals interviewers look for:** you tie stage boundaries to shuffles and tasks to partitions; you use `explain` and the UI.

### 11. Check yourself
- [ ] Given a code snippet, count jobs, stages and tasks.
- [ ] Explain narrow vs wide dependency with two examples each.
- [ ] Say why a "skipped stage" appears in the UI.
- [ ] Describe how a straggler task appears and two ways to fix it.
- [ ] Explain what determines the task count after a shuffle.
<!-- explainer:end -->

## References

- *Hadoop: The Definitive Guide* — Tom White — Spark › DAG Construction
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Downloading Apache Spark and Getting Started › Spark Jobs
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Accelerator-Aware Scheduler
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Project Finding the fastest way to get around NYC › Project: Finding the fastest way to get around NYC

