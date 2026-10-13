---
id: "TOOLS/Apache_Spark/03_Spark_Streaming/Streaming_Reliability"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Streaming Fault Tolerance and Monitoring

**L4 Advanced** · [↑ Structured Streaming](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Structured Streaming recovers from crashes by replaying the last uncommitted micro-batch from checkpointed offsets, and delivers end-to-end exactly-once results only when the source is replayable, the computation deterministic and the sink idempotent.

### 1. The big picture
Streaming jobs run for months, so failures (executor loss, node reboots, deploys, bad records, source outages) are routine. Reliability means: no data loss, no unintended duplicates, and bounded recovery time. Spark's answer combines three ideas: **micro-batching with a write-ahead log** (each batch is a deterministic, replayable unit), **checkpointing of state** (rebuild after failure), and **idempotent output** (re-executed batches don't duplicate). The same reasoning applies across data systems: guarantees hold end to end only if every hop cooperates.

### 2. Core concepts and vocabulary
- **Delivery semantics** — at-most-once (may lose), at-least-once (may duplicate), exactly-once (effect applied once).
- **Replayable source** — can re-read the same offsets (Kafka, files) after a failure; a socket is not.
- **Deterministic computation** — same input range always yields the same output (no random, wall-clock or external lookups that change).
- **Idempotent sink** — re-writing the same batch has no extra effect (batch-id keyed commit, upsert by key, transactional write).
- **Checkpoint** — durable record of query identity, planned offsets, committed batches and state.
- **Write-ahead log (offset log)** — range is recorded *before* processing.
- **Commit log** — written *after* the sink finished the batch.
- **Fault domain** — the thing that can fail: task, executor, driver, source, sink, storage.
- **Recovery time / lag** — how long and how far behind the query is after restart.

### 3. How it works, step by step
1. **Plan batch N:** determine the new range per source partition and write it to `offsets/N` in the checkpoint.
2. **Execute:** run the incremental plan; update state store version N in memory.
3. **Write output** to the sink with the batch id.
4. **Persist state** version N to checkpoint storage (delta files).
5. **Commit:** write `commits/N`.
6. **Failure cases:**
   - *Task/executor failure:* Spark retries tasks; deterministic execution regenerates the same partition; state is reloaded from the last checkpointed version.
   - *Driver/application failure after step 1 but before 5:* on restart, the engine sees `offsets/N` without `commits/N` and **re-runs batch N with the same range**, then continues; the idempotent sink ignores the duplicate write.
   - *Failure after commit:* nothing to redo; start at N+1.
7. **Restart procedure:** same code, same checkpoint location, new `SparkSession` (supervised by YARN/Kubernetes/Airflow/Databricks Jobs with automatic restart).

### 4. Worked example
```python
def upsert_to_warehouse(batch_df, batch_id):
    # idempotent: the table write is keyed by batch_id, so a replay does not duplicate
    (batch_df.withColumn("batch_id", F.lit(batch_id))
             .write.mode("overwrite")
             .option("replaceWhere", f"batch_id = {batch_id}")
             .format("delta").save("/lake/events"))

q = (spark.readStream.format("kafka")
       .option("kafka.bootstrap.servers", "k:9092")
       .option("subscribe", "orders")
       .option("startingOffsets", "earliest")
       .option("failOnDataLoss", "true")
       .option("maxOffsetsPerTrigger", 500_000)
       .load()
     .selectExpr("CAST(key AS STRING) k", "CAST(value AS STRING) v", "timestamp")
     .writeStream
     .foreachBatch(upsert_to_warehouse)
     .option("checkpointLocation", "s3://chk/orders_v1")   # durable, unique per query
     .trigger(processingTime="30 seconds")
     .start())
```
Walkthrough of a crash: batch 41 planned offsets `orders-0: 1000–1500` and wrote half its output when the driver died. On restart Spark finds `offsets/41` with no commit, re-reads exactly `1000–1500`, recomputes (deterministic), and rewrites batch 41 with the same `batch_id`; the replace-by-batch-id write makes this safe. The first progress report shows `batchId: 41` again. Note the word-count example in the Spark docs with a socket source and console sink does *not* give exactly-once: the socket cannot be replayed and the console sink is not idempotent.

### 5. Under the hood
- **Why micro-batch makes it simple:** the unit of recovery is a batch defined by offsets; Spark's deterministic task re-execution regenerates identical output for it. Continuous processing lacks this and gives at-least-once.
- **Checkpoint layout:** `metadata`, `offsets/`, `commits/`, `sources/`, `state/<operator>/<partition>/<version>.delta|.snapshot`. Old versions are cleaned (`spark.sql.streaming.minBatchesToRetain`, default 100).
- **Exactly-once vs effects:** exactly-once is an *output effect* guarantee, not "each record is processed once": records may be recomputed; the sink must absorb it. Side effects inside transformations (HTTP calls, counters) can repeat.
- **Sinks:** file sink uses a metadata log (`_spark_metadata`) so readers see only committed files; Kafka sink is at-least-once (producer retries, duplicates possible) → deduplicate downstream by key; `foreachBatch` gives `batch_id` for idempotency; `foreach` gives `(partitionId, epochId)`.
- **Source specifics:** Kafka offsets in checkpoint; if retention deletes unread data, `failOnDataLoss` decides between failing and continuing. File source records processed file paths (and optionally cleans/archives).
- **Allowed changes on restart:** stateless transformations (filters, projections), adding/removing sinks that are append-only, changing rate limits and triggers are mostly fine; changing aggregation keys, state schema, shuffle partitions, source type or output mode is not.
- **Driver as a single point of failure:** use cluster deploy mode with restart policies and keep checkpoints on durable shared storage (HDFS/S3/ADLS), not local disks.
- **End-to-end argument:** reliability is a property of the whole pipeline; add end-to-end checks (reconciliation counts, unique business keys, audit tables) rather than trusting each layer.

### 6. Performance and design implications
- Recovery time = driver restart + executor startup + state load + replaying the failed batch + catching up the backlog. Smaller batches and `maxOffsetsPerTrigger` reduce replay; RocksDB with incremental checkpoints reduces state-restore cost.
- Checkpoint storage latency is on the critical path (offset log, commit log, state delta each batch): use fast, strongly consistent storage and avoid very short triggers on slow object stores.
- Unbounded retries mask poison messages: parse defensively, route bad records to a dead-letter location, and alert.
- Watch: input vs processed rate, batch duration trend, state size, time since last successful batch, failed restarts, source lag (e.g. Kafka offsets behind).
- Plan for upgrades: version checkpoint paths (`orders_v1`), run old/new in parallel and cut over, or store state-compatible changes only.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `checkpointLocation` | Recovery metadata/state | Always; durable shared storage |
| `.foreachBatch(fn(batch_df, batch_id))` | Idempotent custom sink | JDBC, merge, multi-sink |
| `failOnDataLoss` (Kafka) | Fail vs continue if offsets are gone | Strict vs lenient pipelines |
| `maxOffsetsPerTrigger` / `maxFilesPerTrigger` | Cap batch size | Recovery, burst smoothing |
| `spark.sql.streaming.minBatchesToRetain` | Checkpoint retention | Disk usage, rollback |
| `spark.sql.streaming.stateStore.providerClass` | RocksDB state | Large state, faster recovery |
| `spark.streaming.stopGracefullyOnShutdown` / `query.stop()` | Clean shutdown | Deployments |
| `StreamingQueryListener` | Progress/termination events | Alerting |
| Restart policy (YARN `maxAppAttempts`, K8s `restartPolicy`, scheduler retries) | Re-launch driver | Production |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Exactly-once via idempotent sink for financial/billing pipelines | Non-idempotent external side effects inside the pipeline |
| At-least-once + downstream dedup for Kafka output | Consumers unable to deduplicate |
| `failOnDataLoss=true` for correctness-critical data | Topics with short retention and long outages → stuck query |
| Frequent small batches for low latency | Checkpoint overhead and small files |
| New checkpoint + reprocess for incompatible changes | Reprocessing cost and output duplication risks |

### 9. Common mistakes
- Checkpoint on local disk or `/tmp` → lost on node failure → durable shared storage.
- Sharing or deleting checkpoints casually → duplicates or data loss → one per query, version them.
- Non-deterministic transforms (`rand()`, `current_timestamp()`, external API calls) → replayed batch differs → derive from event data or accept at-least-once.
- Non-idempotent `foreachBatch` (plain `append` to a table) → duplicates after restart → keyed upsert or batch-id replace.
- Ignoring poison pills → infinite restart loop → dead-letter handling.
- No monitoring of lag → silent backlog growth until retention deletes data.

### 10. 🎯 Interview kit
**30-second answer:** Spark writes the planned offsets before processing and a commit marker after, with state checkpointed per batch. After a failure it replays the uncommitted batch from the same offsets deterministically; if the source is replayable and the sink idempotent that yields end-to-end exactly-once.
**Structure a longer answer:** clarify guarantees needed → three conditions for exactly-once → checkpoint/offset/commit mechanics → failure scenarios (task, driver, sink) → operations: restarts, changes, monitoring, poison messages.
**Likely follow-ups:**
- *What if the driver dies mid-batch?* → rerun from offset log with same range.
- *Is Kafka sink exactly-once?* → no; at-least-once, dedupe downstream.
- *What can't you change on restart?* → state schema, keys, partitions, output mode.
- *How do you make a JDBC sink safe?* → upsert/delete+insert by batch id in one transaction.
- *Exactly-once vs effectively-once?* → records may be reprocessed; effects are applied once.
**Signals interviewers look for:** precision on where duplicates come from and who absorbs them, practical experience with restarts, and end-to-end verification habits.

### 11. Check yourself
- [ ] Explain offsets log vs commit log with a failure between them.
- [ ] State the three conditions for exactly-once and give a violating example of each.
- [ ] Write an idempotent `foreachBatch` for a JDBC/warehouse target.
- [ ] List which query modifications are safe on an existing checkpoint.
- [ ] Describe the monitoring you would put on a production streaming job.
<!-- explainer:end -->

## References

- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Generation in Source Systems › Fault tolerance and resilience
- *Designing Data-Intensive Applications* — Martin Kleppmann — Fault Tolerance
- *Designing Data-Intensive Applications* — Martin Kleppmann — Fault Tolerance › Microbatching and checkpointing
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Under the Hood of an Active Streaming Query › Recovering from Failures with Exactly-Once Guarantees
- *Designing Data-Intensive Applications* — Martin Kleppmann — Aiming for Correctness › Applying end-to-end thinking in data systems
- *Database Internals* — Alex Petrov — PBFT Algorithm › Recovery and Checkpointing
- *Designing Data-Intensive Applications* — Martin Kleppmann — Graphs and Iterative Processing › Fault tolerance
- *Designing Data-Intensive Applications* — Martin Kleppmann — Fault Tolerance › Rebuilding state after a failure
- *Database Internals* — Alex Petrov — Transaction Processing and Recovery › Recovery
- *Database Internals* — Alex Petrov — Further Reading › Appendix A. Bibliography

