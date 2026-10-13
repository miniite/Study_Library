---
id: "02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Safe_Re-runs_of_Failed_Jobs"
level: 4
type: "concept"
topic: "02_Data_Engineering/Pipelines_and_Integration/ETL_ELT"
tools: ["Apache_Airflow"]
---

# Safe Re-runs of Failed Jobs

**L4 Advanced** · [↑ ETL ELT](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A job is safe to re-run when its output depends only on its inputs and its logical time window, so running it again after a failure replaces its results instead of duplicating or corrupting them.

### 1. The big picture
Jobs fail: a node dies, an API times out, a warehouse hits a quota, a deploy goes wrong halfway. Orchestrators retry automatically and humans press "re-run". If the job is not designed for that, a retry duplicates rows, double-counts revenue or leaves half-written tables.

Safe re-runs rest on three ideas: **idempotency** (same input, same output regardless of how many runs), **atomicity** (a run's output becomes visible all at once or not at all), and **determinism** (the run is parameterized by a logical window, not by "now"). Combined with observability and a clear recovery procedure, a failed job becomes a routine event.

### 2. Core concepts and vocabulary
- **Idempotent** — running once or many times yields the same end state.
- **Atomic commit** — all-or-nothing visibility of a run's writes.
- **Logical (execution) date / data interval** — the time window the run is responsible for, passed in as a parameter; not the wall-clock time of execution.
- **Overwrite by partition** — replace the entire partition for the window rather than appending.
- **Staging + swap** — write to a temporary table/location, validate, then switch it into place.
- **Checkpoint** — saved progress so work can resume from a known point.
- **Retry policy** — number of attempts, delay, backoff.
- **Poison record** — a record that deterministically causes failure on every attempt.
- **Dead-letter queue (DLQ) / error table** — destination for records that cannot be processed, so the job can proceed.
- **At-least-once delivery** — messages may be delivered more than once, requiring dedupe.
- **Run ID / batch ID** — identifier stamped on outputs to trace and clean up a run.

### 3. How it works, step by step
**Designing a re-runnable batch job**
1. **Parameterize by window**: `run_date` or `[start, end)`; never use `now()` to choose data.
2. **Read a deterministic input slice**: filter source by the window (and a stable snapshot if possible).
3. **Write to a staging location** keyed by run ID.
4. **Validate** (row counts, key uniqueness, null checks).
5. **Commit atomically**: swap/partition-overwrite/`MERGE` in a single transaction or by metadata operation.
6. **Record** run metadata (inputs, outputs, counts, status).
7. **On failure**: nothing was published; re-run the same window.

**Recovering from a failure**
1. Read the logs and classify: transient (network, throttling), resource (OOM, disk), data (bad record, schema change), logic (bug).
2. Fix the cause (retry, resize, quarantine poison record, patch code).
3. Re-run only the failed window(s) and downstream dependents.
4. Validate outputs against expectations.
5. Document the incident and add a test or alert.

### 4. Worked example
Daily load of orders into a partitioned table.

**Unsafe**: appends rows each run.

```sql
INSERT INTO dw.orders SELECT * FROM stg.orders;   -- retry = duplicates
```

**Safe, partition overwrite**:

```sql
-- run_date = '2026-10-11'
BEGIN;
DELETE FROM dw.orders WHERE order_date = DATE '2026-10-11';
INSERT INTO dw.orders
SELECT * FROM stg.orders WHERE order_date = DATE '2026-10-11';
COMMIT;
```

**Safe, merge by key** (late updates allowed):

```sql
MERGE INTO dw.orders t
USING stg.orders_dedup s ON t.order_id = s.order_id
WHEN MATCHED THEN UPDATE SET ...
WHEN NOT MATCHED THEN INSERT ...;
```

Delivered through an orchestrator: tasks receive `{{ ds }}`, retry with backoff, and downstream tasks wait on success. A task that calls an external API passes an **idempotency key** (such as `orders-2026-10-11-batch-3`) so a retried request does not create a second charge or record.

Handling a flaky partner API: set connect/read timeouts, retry transient codes (429, 5xx) with exponential backoff and jitter, cap total attempts, write each page's response to staging with page cursor checkpoints so a retry resumes from the last good page.

### 5. Under the hood
- **Why append + retry duplicates.** The first attempt may have written some or all rows before failing; the retry has no knowledge of that. Overwrite/merge semantics make the write independent of prior attempts.
- **Atomic visibility techniques.** Warehouse transactions; table swap or rename; writing files to a new path then updating a pointer/manifest (the approach table formats like Delta/Iceberg formalize through transaction logs); partition exchange.
- **Exactly-once effects.** True exactly-once delivery is hard; practical systems achieve *effectively once* by combining at-least-once delivery with idempotent writes (keys, upserts, dedupe tables, transactional sinks).
- **Poison records.** A bad record will fail every retry. Isolate: catch per-record exceptions, route to DLQ with context, continue, and alert on the DLQ rate. Alternatively fail the batch intentionally if any bad record is unacceptable (financial data), but then fix the record and re-run.
- **Side effects.** Emails, payments, external API writes are not undone by re-running. Separate computation from effects; make effects idempotent with keys, or record "sent" state transactionally.
- **Dependencies.** Re-running upstream may require re-running downstream tasks whose inputs changed; orchestrators can clear and re-run downstream dependents. Ensure downstream tasks are also idempotent.
- **Time-dependent code.** Using `now()`, random values or non-deterministic ordering inside transforms makes outputs differ between runs; pass time in and order explicitly.

### 6. Performance and design implications
- Partition-level overwrites make re-runs cheap and bounded: only the affected window is recomputed.
- Row-level merges cost more than partition overwrite but handle late updates.
- Retries add load; limit attempts and use backoff so recovery doesn't worsen an outage.
- Keep raw inputs immutable and retained at least as long as the maximum backfill horizon.
- Alert on repeated failure of the same window and on duration anomalies; track mean time to recover.
- Test idempotency: run the job twice in CI and assert identical outputs and counts.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `INSERT OVERWRITE` / partition overwrite / `REPLACE WHERE` | Replace only a window | Daily/hourly partitioned loads |
| `MERGE` / `INSERT ... ON CONFLICT DO UPDATE` | Upsert by key | Mutable entities, late updates |
| Orchestrator retries, `retry_delay`, exponential backoff | Automatic retry policy | Transient failures |
| Orchestrator "clear/re-run task and downstream" | Targeted re-execution | Recovery after fix |
| Idempotency key header/field | De-duplicates external calls | Payments, API writes |
| DLQ / error table | Holds unprocessable records | Poison records |
| Run/batch ID columns | Trace and delete a run's output | Debugging, cleanup |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Partition overwrite: immutable, window-aligned data | Updates arrive for old partitions you haven't included |
| Merge by key: mutable records, late data | No stable unique key |
| Staging + swap: large loads needing validation before publish | Storage engines without cheap atomic swap |
| DLQ and continue | Data where any dropped record is unacceptable |
| Automatic retries | Non-idempotent side effects |

### 9. Common mistakes
- Plain append in a retried task → duplicates → overwrite window or merge.
- Using `current_date` inside SQL → re-run yields different data → pass logical date.
- Marking a run successful before validation → bad data published → validate then commit.
- Retrying a deterministic failure forever → wasted compute → classify, route poison records.
- Re-running a middle task without its dependents → inconsistent downstream tables → re-run downstream chain.
- Non-idempotent external calls → double charges → idempotency keys.

### 10. 🎯 Interview kit
**30-second answer:** I make jobs idempotent and atomic: parameterize by logical window, read a deterministic slice, write to staging, validate, then commit with partition overwrite or merge so a retry replaces rather than duplicates. Poison records go to a DLQ, external calls use idempotency keys, and recovery is re-running the failed window plus downstream.

**Structure a longer answer:** failure classification → idempotent design → atomic publish → retry/backoff → effects and DLQ → validation and monitoring.

**Likely follow-ups:**
- *Why duplicates under at-least-once?* → redelivery or retry; fix by keyed upserts/dedupe.
- *Overwrite vs merge?* → window-immutable data vs mutable records.
- *How do you test idempotency?* → run twice, compare outputs.
- *Backfill a month safely?* → loop over windows with limited concurrency, validate each.

**Signals interviewers look for:** you separate computation from side effects and can state concrete mechanisms for atomicity.

### 11. Check yourself
- [ ] Convert an append-based load into an idempotent one in two ways.
- [ ] List three ways to make a publish atomic.
- [ ] Explain how a DLQ changes failure behavior and what to monitor.
- [ ] Describe a safe retry policy for a rate-limited API.
- [ ] State why `now()` inside a transform breaks re-runs.
<!-- explainer:end -->

## Implementations (tools)

- [Apache Airflow](<../../../TOOLS/Apache_Airflow/_index.md>)

## Practice questions

- [Basic] [How do you decide the frequency (hourly, daily, weekly) of a batch ETL job?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Q-24785_How_do_you_decide_the_frequency_hourly_daily_weekly_of_a_bat.md>)
- [Basic] [Why do we log the start time, end time, and row counts of each ETL run?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Q-24793_Why_do_we_log_the_start_time_end_time_and_row_counts_of_each.md>)
- [Intermediate] [How do you ensure an ETL job that failed halfway can be safely re-run?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Q-24777_How_do_you_ensure_an_ETL_job_that_failed_halfway_can_be_safe.md>)
- [Intermediate] [What could cause an ETL job's runtime to slowly grow over weeks, and how would you investigate?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Q-24819_What_could_cause_an_ETL_job_s_runtime_to_slowly_grow_over_we.md>)
- [Intermediate] [How would you alert the team when an ETL job fails or produces zero rows?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Q-24829_How_would_you_alert_the_team_when_an_ETL_job_fails_or_produc.md>)
- [Intermediate] [How would you make sure a nightly ETL finishes before business hours, and what if it does not?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Q-24845_How_would_you_make_sure_a_nightly_ETL_finishes_before_busine.md>)
- [Deep] [How would you design an ETL job to be restartable from where it left off rather than the beginning?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Q-24838_How_would_you_design_an_ETL_job_to_be_restartable_from_where.md>)
- [Deep] [How do you make an ELT pipeline observable enough that on-call can triage a failure in under ten minutes at 3am?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Q-24874_How_do_you_make_an_ELT_pipeline_observable_enough_that_on-ca.md>)

## Related concepts

- [End to End No Data Loss](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/End_to_End_No_Data_Loss.md>) *(Data Architecture)*
- [Pipeline Design Patterns](<../Pipeline_Design/Pipeline_Design_Patterns.md>) *(Pipeline Design)*
- [Pipeline Data Monitoring](<../Pipeline_Orchestration/Pipeline_Data_Monitoring.md>) *(Pipeline Orchestration)*

## References

- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Infrastructure Operations › ETL pipeline maintenance
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Infrastructure Operations › Debugging the job queue
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — DataOps and AI › Monitoring and continuous improvement at ACME
- *Mapping Data Flows in Azure Data Factory* — Mark Kromer — ETL for the Cloud Data Engineer › Typical SDLC
- *Mapping Data Flows in Azure Data Factory* — Mark Kromer — Monitor, Manage, and Optimize › Time to Live (TTL)
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Backups › 12.5 #82 Taking backups during an ETL window
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 11: Automation with SSIS, SQL Agent, and PowerShell › Logging and Auditing
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 11: Automation with SSIS, SQL Agent, and PowerShell › Automated Testing in Pipelines
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Using Time Travel › How Often Should You Run VACUUM and Other Maintenance Tasks?

