---
id: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Checkpointing_and_Recovery"
level: 4
type: "concept"
topic: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design"
tools: []
---

# Checkpointing and Recovery

**L4 Advanced** · [↑ Pipeline Design](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A checkpoint records how far a pipeline got, so after a failure it resumes from that point instead of starting over — and it is only safe when re-running the work after the checkpoint is idempotent.

### 1. The big picture
Pipelines fail: networks drop, a node is preempted, a source API rate-limits you, a bad row crashes a transform. What matters is how much work you lose and whether the recovery can damage your data. There are three recovery tools that people often blur together:

- **Retry** — run the same failed task again, soon, in the same logical run. It covers transient faults.
- **Checkpoint / resume** — remember progress (an offset, a watermark, completed partitions) so a restarted job continues from there.
- **Backfill / reprocess** — deliberately run historical periods again, because data was missed or logic changed.

Checkpointing sits between the first two: it makes a retry *cheap* (no repeat of finished work) and, combined with idempotent writes, *safe*. Without it, a failure at hour 5 of a 6-hour job costs 5 hours; without idempotency, the re-run may double-count.

### 2. Core concepts and vocabulary
- **Checkpoint** — durable record of pipeline progress (offsets, high-water mark, list of finished partitions, operator state).
- **High-water mark / watermark (batch sense)** — the largest timestamp or ID successfully processed; the next run starts after it.
- **Offset** — position in a log/stream (Kafka offset, file position, CDC log sequence number).
- **Idempotency** — performing an operation multiple times has the same effect as performing it once.
- **At-least-once / at-most-once / exactly-once** — delivery/processing guarantees; exactly-once in practice = at-least-once + idempotent (or transactional) writes.
- **Retry with backoff** — re-attempt after increasing delays to ride out transient failures.
- **Staging table** — an intermediate table where results land before being promoted to the final table atomically.
- **Reprocessing** — recomputing a whole period or dataset from source, as opposed to resuming a failed task.

### 3. How it works, step by step
1. **Break the work into units** that can be tracked independently: a date partition, a file, a key range, a Kafka offset range, a stage of the DAG.
2. **Do the unit of work** and write its output to a staging location or to a deterministic target (e.g. `partition=2026-10-11`).
3. **Commit the output atomically** (rename a staging directory, `MERGE`, swap a partition, commit a transaction).
4. **Then advance the checkpoint** (write the new offset/watermark/"done" marker). Order matters: output first, checkpoint second.
5. **On failure**, the new attempt reads the checkpoint, skips completed units, and re-does only the unit that was in flight.
6. **Because a crash can happen between steps 3 and 4**, the in-flight unit may be re-run even though its output was already written. That is why step 2–3 must be idempotent: the second run overwrites or merges, never appends duplicates.
7. **Escalate**: bounded retries with backoff for transient errors; alert and stop for deterministic failures (bad schema, missing source); never retry forever.
8. **When the logic or upstream data changes**, do not "resume" — run a backfill of the affected period, using the same idempotent write path.

### 4. Worked example
A daily pipeline loads orders for date `D` from an API into a warehouse table.

```sql
-- 1. Stage the day's extract (re-runnable: truncate first)
TRUNCATE TABLE stg.orders_day;
COPY INTO stg.orders_day FROM @landing/orders/2026-10-11/;

-- 2. Idempotent promote: replace exactly that day, in one transaction
BEGIN;
DELETE FROM dw.orders WHERE order_date = DATE '2026-10-11';
INSERT INTO dw.orders SELECT * FROM stg.orders_day;
COMMIT;

-- 3. Only now advance the checkpoint
UPDATE etl.run_state
   SET last_loaded_date = DATE '2026-10-11'
 WHERE pipeline = 'orders_daily';
```

Walk through failures: if the job dies during step 1, the checkpoint still says `2026-10-10`; the re-run truncates and re-stages — no harm. If it dies after step 2's `COMMIT` but before step 3, the re-run repeats step 2: delete-then-insert the same day — still correct. The checkpoint can lag the data but never lead it. Contrast with a plain `INSERT` append: the same crash would double the day's rows.

### 5. Under the hood   *(optional at L4)*
- **Where checkpoints live:** a control table in the warehouse, a file in object storage, the orchestrator's metadata DB (task state per logical date in Airflow), or the engine itself (Spark Structured Streaming's checkpoint directory holds offsets and state-store snapshots; Flink writes distributed snapshots).
- **Granularity trade-off:** frequent checkpoints = little lost work but more overhead; infrequent = cheap but long replays. In streaming engines, checkpoint interval is tuned this way.
- **State vs position:** a stateless job only needs offsets; a stateful aggregation also needs its state saved, or recovery recomputes it from an earlier point.
- **Failure window:** the crash between "output committed" and "checkpoint advanced" is the reason the guarantee is at-least-once; making the sink idempotent or transactional closes it.

### 6. Performance and design implications
- Smaller units (per partition rather than per job) mean cheaper recovery and better parallelism but more metadata to track.
- Idempotent writes cost something (a delete + insert or a merge instead of an append) — worth it, because correctness under retry is not optional.
- Store checkpoints somewhere at least as durable as the data; a checkpoint on a node's local disk is no checkpoint.
- Monitor: age of last checkpoint, retry counts, and the gap between source high-water mark and checkpoint (lag).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| Orchestrator `retries`, `retry_delay`, exponential backoff | Re-run a failed task automatically | Transient faults (network, rate limit) |
| Control/state table with `last_loaded_*` | Persist batch progress | Incremental loads |
| `checkpointLocation` (Spark Structured Streaming) | Persist offsets and state | Any streaming query that must resume |
| `MERGE` / partition overwrite | Idempotent promotion | Writing results so re-runs are safe |
| Backfill command / `clear` of past runs | Re-run historical periods | Logic fixes, late data |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Resume from checkpoint — failure was transient and logic is unchanged | Source data or logic changed; a resume would mix old and new results |
| Fine-grained checkpoints — long jobs, expensive steps | Tiny units make bookkeeping heavier than the work |
| Full reprocess — correctness matters more than time, data is small | Data is huge and only one partition is wrong |
| Retry — error is transient | Error is deterministic (schema break): retries just delay the alert |

### 9. Common mistakes
- Advancing the checkpoint *before* the output is durable → data silently lost on crash → write output first, checkpoint second.
- Append-only writes with retries enabled → duplicates → make the write idempotent (overwrite partition or merge on key).
- Confusing retry timeouts with retries (just waiting longer) → no real recovery → configure actual re-execution.
- Not logging retry attempts → impossible to audit → log attempt number, error, and checkpoint value.
- No backfill plan → gaps stay forever → keep the same code path for backfill and daily runs, parameterized by date.

### 10. 🎯 Interview kit
**30-second answer:** A checkpoint is durable progress — an offset or watermark — so a failed job resumes instead of restarting. I write output atomically first, then advance the checkpoint, and make the write idempotent, because a crash between them causes a re-run. Retries handle transient faults; backfills handle missed or changed data.

**Structure a longer answer:** clarify (batch or streaming, state, sink) → what the unit of work is → commit-then-checkpoint ordering → idempotency → retries vs backfill → monitoring.

**Likely follow-ups:**
- *Retry vs backfill?* → retry is the same run again after a failure; backfill is intentional reprocessing of a period.
- *How do you get exactly-once?* → at-least-once delivery plus idempotent or transactional sink.
- *Where do you store checkpoints?* → durable shared store, often next to the data or in orchestrator metadata.
- *What if the checkpoint is lost?* → fall back to a safe earlier point and rely on idempotent writes, or re-derive progress from the target (max loaded date).

**Signals interviewers look for:** ordering of commit and checkpoint, idempotency, distinguishing recovery mechanisms, and awareness that logic changes require reprocessing, not resuming.

### 11. Check yourself
- [ ] Draw the timeline of a crash between output commit and checkpoint update and say what the re-run does.
- [ ] Rewrite an append-only load so it is idempotent.
- [ ] Explain retry, checkpoint resume, and backfill in one sentence each.
- [ ] Decide what unit of checkpointing you would use for a 10 TB nightly job.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is parameterization in a pipeline and why does it help with reuse?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Q-12881_What_is_parameterization_in_a_pipeline_and_why_does_it_help_.md>)
- [Intermediate] [What does idempotency mean in a data pipeline and why is it useful when a job re-runs?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Q-12843_What_does_idempotency_mean_in_a_data_pipeline_and_why_is_it_.md>)
- [Intermediate] [What is the difference between a retry and a backfill in the context of pipelines?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Q-12847_What_is_the_difference_between_a_retry_and_a_backfill_in_the.md>)
- [Intermediate] [What is the difference between a landing zone, a raw zone, and a curated zone in a data lake pipeline?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Q-12870_What_is_the_difference_between_a_landing_zone_a_raw_zone_and.md>)
- [Intermediate] [What is data freshness and how do you make sure downstream users know how fresh the data is?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Q-12914_What_is_data_freshness_and_how_do_you_make_sure_downstream_u.md>)
- [Intermediate] [What is the difference between reprocessing data and simply re-running a failed task?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Q-12916_What_is_the_difference_between_reprocessing_data_and_simply_.md>)
- [Intermediate] [What is a read replica and how could a pipeline use one for extraction?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Q-12934_What_is_a_read_replica_and_how_could_a_pipeline_use_one_for_.md>)
- [Intermediate] [What is chunking or batching within a pipeline and why would you process data in chunks?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Q-12936_What_is_chunking_or_batching_within_a_pipeline_and_why_would.md>)

## Related concepts

- [Pipeline Design Patterns](<Pipeline_Design_Patterns.md>) *(Pipeline Design)*
- [Cost of Data Cleaning](<../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Cost_of_Data_Cleaning.md>) *(Data Preprocessing)*
- [Batch and Stream Reconciliation](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Batch_and_Stream_Reconciliation/_index.md>) *(Batch vs Stream)*

## References

- *Data Pipelines Pocket Reference* — James Densmore — Introduction to Data Pipelines › Chapter 1. Introduction to Data Pipelines
- *Data Pipelines Pocket Reference* — James Densmore — Introduction to Data Pipelines › What Are Data Pipelines?
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Why Data Quality Deserves Attention---Now › Streaming data
- *Data Algorithms with Spark* — Mahmoud Parsian — **CHAPTER 12** (cont.)
- *Building Data Integration Solutions* — Jay Borthen — Key Concepts in Data Integration › Conditioning
- *Building Data Integration Solutions* — Jay Borthen — Key Terms Glossary

