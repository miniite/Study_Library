---
id: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Handling_Poison_Records"
level: 4
type: "concept"
topic: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration"
tools: ["Apache_Spark", "Apache_Airflow", "Python", "AWS", "Hadoop"]
---

# Handling Poison Records

**L4 Advanced** · [↑ Pipeline Orchestration](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A poison record is a single bad input that keeps crashing a job; you handle it by catching it at the edge, routing it to a quarantine with its error, and letting the good data keep flowing.

### 1. The big picture
Distributed jobs process millions of records in parallel, but a job is only as strong as its weakest record. One malformed JSON line, one value that overflows a cast, or one truncated file can fail a single task. Spark retries a failed task (4 times by default via `spark.task.maxFailures`), the same record fails again on every attempt, and eventually the whole stage and job abort. In a streaming job the effect is worse: the query restarts from its checkpoint, re-reads the same offsets, hits the same record, and loops forever while lag grows. That is a **poison record** (or "poison pill" in messaging systems).

Handling them is part of the error-management layer of a pipeline, next to retries, checkpoints, and late-data handling. The goal is not "never fail" but **fail narrowly**: isolate the bad record, keep it (never silently drop it), alert on it, and let the rest of the batch complete. This sits under pipeline orchestration because the orchestrator decides what happens next — retry the task, skip, page someone, or block downstream.

### 2. Core concepts and vocabulary
- **Poison record** — an input that deterministically causes processing to fail, so retrying never helps.
- **Corrupt file vs corrupt record** — a *file* may be unreadable (bad Parquet footer, truncated gzip); a *record* is a bad row inside an otherwise fine file. They are handled by different options.
- **Parse mode** — how Spark's file readers treat malformed rows: `PERMISSIVE` (default; keep the row, set bad fields to null), `DROPMALFORMED` (silently discard), `FAILFAST` (abort on first bad row).
- **Corrupt-record column** — in `PERMISSIVE` mode, a string column (default name `_corrupt_record`) that receives the raw text of rows that failed to parse.
- **Dead-letter queue (DLQ) / quarantine** — a side location (topic, table, or folder) where rejected records are stored together with the reason, source, and timestamp.
- **Checkpoint** — persisted progress (offsets, state) that lets a job resume after failure without reprocessing everything; it is also why a poison record in a stream gets re-read after each restart.
- **Replay** — reprocessing quarantined records after the bug or data issue is fixed.
- **Bad-record threshold** — a rule such as "fail the job if more than 1% of rows are rejected", separating normal noise from a systemic upstream break.

### 3. How it works, step by step
A robust pattern has the same shape in batch and streaming:

1. **Read defensively.** Read the source with an explicit schema (not inference) and a tolerant parse mode, or read the raw payload as a string/bytes and parse in a controlled step.
2. **Validate and parse inside a guarded step.** Wrap parsing or risky logic so an exception becomes *data*: a row with `error_reason` filled in instead of a thrown exception.
3. **Split the stream of rows** into `good` and `bad` using a flag.
4. **Write good rows** to the normal target.
5. **Write bad rows** (raw payload + error + source file/offset + processing time) to the quarantine.
6. **Measure.** Compute `bad / total`; emit it as a metric; fail or alert if it crosses the threshold.
7. **Triage and replay.** A human or an automated job fixes the cause, then reprocesses quarantined rows through the same code path (idempotently, so replay can't duplicate).

For *corrupt files*, Spark has `spark.sql.files.ignoreCorruptFiles` and `ignoreMissingFiles`: with them set to `true` the job skips unreadable files — convenient but dangerous, because skipped files are lost silently unless you log them. Prefer to list and quarantine the file explicitly.

On Databricks the reader option `badRecordsPath` writes rejected rows and files (with exception messages) to a folder as JSON and continues, which gives you a built-in DLQ for file sources.

### 4. Worked example
```python
from pyspark.sql import functions as F
from pyspark.sql.types import *

schema = StructType([
    StructField("user_id", LongType()),
    StructField("amount",  DecimalType(12, 2)),
    StructField("ts",      TimestampType()),
    StructField("_corrupt_record", StringType()),   # must be in schema to be filled
])

raw = (spark.read
       .schema(schema)
       .option("mode", "PERMISSIVE")
       .option("columnNameOfCorruptRecord", "_corrupt_record")
       .json("s3://lake/raw/orders/2026-10-12/"))

raw = raw.cache()          # Spark refuses queries touching ONLY _corrupt_record on raw files unless cached

good = raw.filter(F.col("_corrupt_record").isNull()).drop("_corrupt_record")
bad  = (raw.filter(F.col("_corrupt_record").isNotNull())
           .select(F.col("_corrupt_record").alias("payload"),
                   F.input_file_name().alias("source_file"),
                   F.current_timestamp().alias("quarantined_at"),
                   F.lit("json_parse_error").alias("reason")))

good.write.mode("append").saveAsTable("silver.orders")
bad.write.mode("append").saveAsTable("quarantine.orders")

total, n_bad = raw.count(), bad.count()
if total and n_bad / total > 0.01:
    raise RuntimeError(f"Bad-record ratio {n_bad/total:.2%} exceeds 1% threshold")
```
Walk-through: the explicit schema stops inference from guessing. `PERMISSIVE` keeps every row; malformed ones carry their raw text in `_corrupt_record`. Rows are split, both sides persisted, and a ratio check turns "a few bad rows" (normal) into "the source changed format" (alert). Note the cache: since Spark 2.3, queries that reference only the corrupt-record column of a raw JSON/CSV source are disallowed to avoid ambiguous results, so you cache or select other columns too.

For a streaming job, do the split inside `foreachBatch`, writing `good` and `bad` in the same micro-batch function; keep `checkpointLocation` on durable storage so progress survives restarts.

### 5. Under the hood
- **Why retries don't help:** a task failure from deterministic data is reproducible; `maxFailures` just burns time. Retries are only useful for transient faults (lost executor, network blip, spot-instance loss).
- **Exactly-once and replay:** Structured Streaming recovers by re-reading the offset range written in its write-ahead log for the last incomplete batch. This works because sources are *repeatable* (Kafka, object storage). It also means a poison record is guaranteed to be delivered again — the fix must be in code or data, not in restarting.
- **Python UDFs:** an exception raised inside a UDF surfaces as a task failure wrapped in a Python traceback. Catch inside the UDF and return a struct `(value, error)` instead.
- **Event-time/partition trackers:** unrelated but often confused — a "stuck in the past" stateful job isn't a poison record; it's a watermark issue where one stale partition holds back the minimum event time.

### 6. Performance and design implications
- Quarantine writes add a second output; keep them small by storing raw payload + reason, not full enriched rows.
- `cache()`/double scans of the input cost memory or I/O; with columnar files prefer a single pass producing a flag column.
- Counting twice (`raw.count()`, `bad.count()`) triggers actions; compute counts with one `groupBy(flag).count()` or via accumulators/metrics.
- Many tiny quarantine files (one per micro-batch) create the small-file problem — compact them periodically.
- Measure: bad ratio per source per run, quarantine growth, time-to-triage.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `mode=PERMISSIVE / DROPMALFORMED / FAILFAST` | Parse-mode for CSV/JSON readers | PERMISSIVE + corrupt column for quarantine; FAILFAST when any bad row means upstream is broken |
| `columnNameOfCorruptRecord` | Names the raw-text column | Capturing bad rows |
| `badRecordsPath` (Databricks) | Writes rejects + errors to a folder | File sources needing zero-code DLQ |
| `spark.sql.files.ignoreCorruptFiles` | Skip unreadable files | Only with logging; otherwise data loss |
| `spark.task.maxFailures` | Retry count per task | Raise for flaky infra, not for bad data |
| `checkpointLocation` | Streaming progress store | Always for streaming |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `FAILFAST` — data must be perfect (finance close) | Sources with routine noise; one row would stop everything |
| `PERMISSIVE` + quarantine — default for ingestion | Nobody reviews the quarantine; it becomes a silent graveyard |
| `DROPMALFORMED` — telemetry where loss is acceptable | Anything auditable; you can't prove what was dropped |
| Threshold-based failure — mixed tolerance | Threshold too loose hides a real schema break |

### 9. Common mistakes
- Dropping bad rows silently → counts don't reconcile and nobody knows → always write rejects with a reason and count them.
- Retrying forever → a stream stuck on one offset while lag grows → route to DLQ after N attempts.
- Treating a corrupt *file* like a corrupt *record* → whole partition lost → quarantine at file level with the path.
- Rejecting the *whole micro-batch* for one row → latency spike → split per row.
- No replay path → quarantine fills and is ignored → build a "reprocess quarantine" job with idempotent writes.

### 10. 🎯 Interview kit
**30-second answer:** Don't let one bad record abort the job. Read with an explicit schema in a tolerant mode, catch parse/validation errors into a flag, write good rows to the target and bad rows plus the reason to a dead-letter location, and alert if the bad ratio crosses a threshold. Keep checkpoints and make replay idempotent.

**Structure a longer answer:** clarify (batch or stream, tolerance for loss, SLA) → core mechanism (parse mode, corrupt column, split, quarantine) → trade-offs (FAILFAST vs PERMISSIVE vs DROPMALFORMED) → real scenario (an upstream app deploy changes a field type overnight).

**Likely follow-ups:**
- *Why not just retry?* → deterministic failures don't recover; retries are for transient faults.
- *What are the correctness implications of the three modes?* → FAILFAST is exact but brittle; PERMISSIVE preserves everything but yields nulls downstream; DROPMALFORMED loses data silently.
- *How do you not lose records you must not drop?* → quarantine with raw payload and replay after a fix.
- *How do you handle it in Kafka?* → deserialize in a guarded step, publish failures to a DLQ topic with headers describing the error.

**Signals interviewers look for:** separation of transient vs deterministic failure, explicit quarantine design, metrics and thresholds, idempotent replay.

### 11. Check yourself
- [ ] Explain why a poison record loops a streaming job forever.
- [ ] Write the split of good/bad rows from a PERMISSIVE read.
- [ ] State what each parse mode does to correctness.
- [ ] Describe how you'd replay quarantined rows without duplicates.
<!-- explainer:end -->

## Implementations (tools)

- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)
- [Apache Airflow](<../../../TOOLS/Apache_Airflow/_index.md>)
- [Python](<../../../TOOLS/Python/_index.md>)
- [AWS](<../../../TOOLS/AWS/_index.md>)
- [Hadoop](<../../../TOOLS/Hadoop/_index.md>)

## Practice questions

- [Intermediate] [What does out-of-memory mean in a Spark job and what beginner-level things could cause it?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13440_What_does_out-of-memory_mean_in_a_Spark_job_and_what_beginne.md>)
- [Intermediate] [What's your approach to unit-testing the Spark noise-scoring transformation without a full cluster and where does local testing miss real issues?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13499_What_s_your_approach_to_unit-testing_the_Spark_noise-scoring.md>)
- [Intermediate] [Explain how you'd checkpoint a long Spark job so a mid-pipeline failure doesn't force recomputation from scratch.](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13166_Explain_how_you_d_checkpoint_a_long_Spark_job_so_a_mid-pipel.md>)
- [Deep] [How do you handle a corrupt or malformed alert payload that fails one Spark task without aborting the whole enrichment job you need for live paging?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13476_How_do_you_handle_a_corrupt_or_malformed_alert_payload_that_.md>)
- [Deep] [How would you eliminate or accelerate a Python UDF that dominates the runtime of your log-parsing Spark job feeding anomaly detection?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13485_How_would_you_eliminate_or_accelerate_a_Python_UDF_that_domi.md>)
- [Deep] [How do you debug a Spark job that runs clean on the cluster but produces wrong noise scores only for a specific edge-case service-name range?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13541_How_do_you_debug_a_Spark_job_that_runs_clean_on_the_cluster_.md>)
- [Deep] [How do you handle a Spark job on spot/preemptible nodes so mid-stage node loss doesn't restart the whole job and what checkpointing enables that?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13742_How_do_you_handle_a_Spark_job_on_spot_preemptible_nodes_so_m.md>)
- [Deep] [How would you handle a corrupt input file that fails one Spark task and aborts a whole feature job you can't afford to lose?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13177_How_would_you_handle_a_corrupt_input_file_that_fails_one_Spa.md>)

## Related concepts

- [Spark Execution Caching and Debugging](<../../../TOOLS/Apache_Spark/Spark_Core/Spark_Execution_Caching_and_Debugging.md>) *(Spark Core)*
- [Streaming Recovery and Deduplication](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Streaming_Recovery_and_Deduplication.md>) *(Big Data Processing)*
- [Spark Executor Sizing](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Spark_Executor_Sizing.md>) *(Big Data Processing)*

## References

- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — E › F
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Processing Incremental Data › Structured Streaming Guarantees
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — E › F
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ingestion Design Patterns › Examples
- *Data Engineering Design Patterns* — Bartosz Konieczny — Error Management Design Patterns › Problem
- *Data Engineering Design Patterns* — Bartosz Konieczny — Error Management Design Patterns › Consequences
- *Data Engineering Design Patterns* — Bartosz Konieczny — Chapter 10. Data Observability Design Patterns › Problem
- *Mapping Data Flows in Azure Data Factory* — Mark Kromer — Common ETL Pipeline Practices in ADF with Mapping Data Flows › Task 2: Debug the Pipeline
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Automated Data Quality Checks and Error Handling › Throttling errors
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Amazon S3 Tables › GenAI-Powered Troubleshooting for Spark in AWS Glue
- *Databricks Certified Associate Developer for Apache Spark* — Saba Shah — What is Apache Spark › Resilient datasets/fault tolerance

