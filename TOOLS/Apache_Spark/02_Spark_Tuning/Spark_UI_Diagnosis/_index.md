---
id: "TOOLS/Apache_Spark/02_Spark_Tuning/Spark_UI_Diagnosis"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Reading the Spark UI

**L4 Advanced** · [↑ Spark Tuning](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** The Spark UI turns an opaque slow job into evidence — jobs and stages show where time goes, task metrics reveal skew/spill/GC, the SQL tab shows the plan with real row counts, and Storage and Environment confirm what was cached and configured.

### 1. The big picture
Tuning without measurement is guessing. Spark exposes a web UI (default port 4040 on the driver while the application runs; the **History Server** serves it afterwards from event logs) that records every job, stage, task, executor and SQL execution. The skill is moving from symptom ("job is slow", "executor lost") to cause (skew, spill, too few partitions, bad join strategy, cache not used) by reading the right tab. This lesson is the diagnostic counterpart to the tuning lessons on parallelism, joins and caching.

### 2. Core concepts and vocabulary
- **Job / stage / task** — action → job → stages split at shuffles → one task per partition.
- **Event timeline / DAG visualisation** — the Jobs and Stages pages draw stages and shuffle boundaries.
- **Summary metrics** — min/25th/median/75th/max per task for duration, GC time, input, shuffle read/write, spill.
- **Shuffle read/write** — bytes moved between stages.
- **Spill (memory/disk)** — execution data that did not fit memory and was written to disk; memory spill is the deserialised size, disk spill the serialised size.
- **GC time** — time tasks spent in garbage collection.
- **Scheduler delay / task deserialisation / executor compute / result serialisation** — components of task time.
- **Straggler** — task far slower than the median.
- **Event log** — JSON log (`spark.eventLog.enabled`, `spark.eventLog.dir`) used by the History Server.

### 3. How it works, step by step
A reliable diagnostic walk-through:
1. **Jobs tab** — find the slow job(s); check the event timeline for gaps (idle cluster, executors being added) and failed jobs.
2. **Stages tab** — open the slowest stage. Read duration, input and shuffle read/write sizes, number of tasks.
3. **Summary metrics for tasks** — compare median and max duration and input size. A max many times the median means **skew**. Non-zero **spill** means too little memory per task or too-large partitions. High GC time means heap pressure.
4. **Task table** — sort by duration to find the straggler; note its executor, locality level, attempts and failure reason.
5. **SQL tab** — open the query: the plan graph shows each operator with row counts and sizes. Look for a scan that read far more than expected (no pushdown), a `SortMergeJoin` where a `BroadcastHashJoin` was possible, `Exchange` volumes, and `CartesianProduct`.
6. **Storage tab** — confirm cached DataFrames: storage level, fraction cached, size in memory/on disk.
7. **Executors tab** — check per-executor tasks, GC time, shuffle, failed tasks; a dead or lost executor shows here with a reason.
8. **Environment tab** — verify the configs that actually took effect (resource settings, AQE, shuffle partitions, Spark version, classpath).
9. **Structured Streaming tab** (streaming jobs) — input rate vs process rate, batch duration, operation durations, state size.

### 4. Worked example
Symptom: a nightly join job takes 55 minutes instead of 10.

```python
spark.conf.set("spark.eventLog.enabled", "true")      # also configure spark.eventLog.dir
df = orders.join(users, "user_id").groupBy("state").count()
df.write.mode("overwrite").parquet("/out/orders_by_state")
```
1. Jobs tab: one job at 54 minutes, three stages. Stage 2 accounts for 50 minutes.
2. Stage 2 summary metrics: median task 12 s, max task 48 min; shuffle read median 80 MB, max 9 GB; spill (disk) of 30 GB only on the max task.
3. Diagnosis: key skew — one `user_id` (a placeholder `0` for guests) holds ~40% of rows, so one task processes 9 GB and spills.
4. SQL tab confirms `SortMergeJoin` on `user_id`, with a skewed `Exchange` partition.
5. Fixes in order of cost: enable AQE skew handling (`spark.sql.adaptive.enabled`, `spark.sql.adaptive.skewJoin.enabled`); or filter/separately process the guest key; or salt the key. After the change, max task ≈ median, job ≈ 9 minutes.

The same method for other symptoms:

| Symptom in UI | Likely cause | Typical fix |
|---|---|---|
| One huge task, others tiny | Data skew | AQE skew join, salting, isolate hot key |
| All tasks spill to disk | Partitions too big / low memory per task | More shuffle partitions, more memory, fewer cores per executor |
| Few tasks, idle cores | Too few partitions | `repartition`, lower `maxPartitionBytes` |
| Tens of thousands of ~50 ms tasks | Too many tiny partitions/files | `coalesce`, compact files, AQE coalesce |
| High GC time | Heap pressure from objects/caching | Serialised/off-heap cache, fewer cached data, bigger heap, Kryo |
| Executor lost / OOM killed | Memory overhead, huge partition, broadcast | Raise overhead/memory, reduce partition size |
| Cache shows low "fraction cached" | Not enough storage memory | Different storage level, cache less |
| Scan reads whole table | No pushdown/partition pruning | Filter on raw columns, partition layout, avoid UDF on filter column |

### 5. Under the hood
- **Event bus.** The driver's listener bus posts events (job start, stage completed, task end with metrics) to the UI and the event-log writer; the History Server replays the log. Very large applications can drop events if the listener queue overflows (`spark.scheduler.listenerbus.eventqueue.capacity`).
- **Task metric breakdown.** Task time = scheduler delay + deserialisation + executor run (which includes input read, compute, shuffle read wait, GC, spill I/O) + result serialisation + getting result. A large scheduler delay points at the driver or resource shortage; large "shuffle read blocked time" points at network or skew in upstream writers.
- **Locality levels** — PROCESS_LOCAL, NODE_LOCAL, RACK_LOCAL, ANY — show whether tasks ran near the data; `spark.locality.wait` trades waiting for locality against idle slots.
- **Stage retries.** FetchFailed errors cause the parent stage to be partially re-run (visible as repeated attempts) — a sign of lost executors/shuffle files or overloaded shuffle service.
- **SQL metrics** include scan "number of output rows", "size of files read", "time spent in scan", sort peak memory, spill size, number of broadcast rows — more precise than stage-level numbers.
- **AQE** shows in the SQL tab as `AdaptiveSparkPlan` with initial and final plans; skew splits and coalesced partitions are annotated on the shuffle reader nodes.

### 6. Performance and design implications
- Keep event logging on in production and run a History Server so post-mortems are possible after the cluster is gone.
- Build the habit: stage duration → task skew → spill/GC → plan. Change one thing at a time and compare.
- Use consistent test data sizes; compare shuffle bytes and spill, not just wall-clock time (which is noisy on shared clusters).
- Add custom metrics/accumulators and structured logs for business-level progress; the UI shows the engine, not your data quality.
- For streaming, alert on input rate > processing rate, rising batch duration and state-store size growth.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `http://driver:4040` | Live UI (4041+ if port busy) | Running applications |
| `spark.eventLog.enabled`, `spark.eventLog.dir` | Persist events | Always in production |
| History Server (`sbin/start-history-server.sh`, `spark.history.fs.logDirectory`) | Browse finished apps | Post-mortems |
| `df.explain("formatted")`, `EXPLAIN COST` | Plan with stats | Pair with SQL tab |
| `spark.sparkContext.setJobDescription("…")`, `setJobGroup` | Label jobs | Find your stage in the UI |
| `spark.ui.retainedJobs/Stages` | UI retention | Large apps |
| `yarn logs -applicationId …` / executor stderr link | Container logs | Executor crashes, OOM |
| `spark.sql.adaptive.enabled` | AQE | See plan changes at runtime |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Live UI for interactive debugging | The app is short-lived; rely on event logs and History Server |
| Task summary metrics to find skew | Few tasks → quantiles meaningless; look at the task table |
| SQL tab for plan-level problems | RDD code has no SQL page; use stage DAG and lineage |
| Executor logs for crashes | Aggregated cluster logs can lag; check container exit codes first |
| Instrumenting with job descriptions | Excess labels add noise; label major stages only |

### 9. Common mistakes
- Judging by total runtime only → misses where the time goes → inspect the slowest stage.
- Looking at average task time → hides skew → compare median vs max.
- Tuning configs without checking the Environment tab → settings silently overridden (or set after the session started) → verify.
- Ignoring "memory spill" because the job finished → hidden slowness → size partitions/memory.
- Forgetting event logs → no data after failure → enable before running.
- Treating lost executors as random → often YARN killing for memory overhead → check exit status 143/137 and raise `memoryOverhead`.

### 10. 🎯 Interview kit
**30-second answer:** I start in the Spark UI: find the slow job and stage, compare median and max task time to detect skew, check spill and GC for memory pressure, and open the SQL tab to see the join strategy, scans and shuffle sizes. Then I fix the specific cause — partitions, skew handling, broadcast, caching, or memory — and re-measure.
**Structure a longer answer:** clarify the symptom and when it started → Jobs → Stages → task distribution → SQL plan → hypothesis → targeted change → verification with the same metrics.
**Likely follow-ups:**
- *How do you detect skew?* → huge max vs median task duration/input, one long task in the timeline.
- *What is spill and why does it hurt?* → execution data written to disk when it exceeds memory; extra I/O and serialisation.
- *Job is done but the UI is gone?* → event logs and History Server.
- *Where to see whether the cache is used?* → Storage tab, `InMemoryTableScan` in the SQL tab.
- *What does a FetchFailed mean?* → lost shuffle output; executor died or shuffle service issue.
**Signals interviewers look for:** a repeatable method, metrics-based claims, familiarity with each tab, and the ability to connect symptoms to specific fixes.

### 11. Check yourself
- [ ] Walk through Jobs → Stages → Tasks → SQL for a slow job and say what each page answers.
- [ ] Given summary metrics (median 10 s, max 20 min), state the hypothesis and two fixes.
- [ ] Explain the difference between memory spill and disk spill figures.
- [ ] Enable event logging and describe how to view a finished application.
- [ ] Find the evidence in the UI that an AQE-optimised query changed its join or partitions.
<!-- explainer:end -->

## References

- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Downloading Apache Spark and Getting Started › The Spark UI
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Optimizing and Tuning Spark Applications › Jobs and Stages
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Optimizing and Tuning Spark Applications › Inspecting the Spark UI
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Optimizing and Tuning Spark Applications › SQL
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Optimizing and Tuning Spark Applications › Environment
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Optimizing and Tuning Spark Applications › Debugging Spark applications
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Optimizing and Tuning Spark Applications › Storage
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Spark SQL and DataFrames Interacting with External Data Sources › Windowing
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — The Fundamentals of a Structured Streaming Query › Putting it all together
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Accelerator-Aware Scheduler › Changed Functionality

