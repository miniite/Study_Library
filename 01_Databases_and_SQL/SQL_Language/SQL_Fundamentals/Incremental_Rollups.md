---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Incremental_Rollups"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: []
---

# Incremental Rollups

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** An incremental rollup processes only the *new or changed* slice of data (tracked by a watermark or partition) and merges it into a pre-aggregated table idempotently, so each run costs O(new data) instead of O(all data) and a re-run never double-counts.

### 1. The big picture
Raw telemetry and event tables grow forever. Dashboards and feature pipelines don't need every raw row, they need *daily/hourly aggregates*. Recomputing the aggregate from scratch each night gets slower every day (and eventually exceeds its window). The alternative is **incremental processing**: keep a rollup table (e.g. per host per day), and each run aggregate only data since the last run, then fold it in.

Three ideas make that safe:
1. **A watermark** (high-water mark) that says how far we've processed.
2. **Idempotent merge** so a restart or re-run doesn't double-count.
3. **Reconciliation** so you can prove the rollup equals the raw data.

Full refresh vs incremental (or CDC) is the central trade-off: full refresh is simple and self-healing but expensive; incremental is cheap but must handle late data, corrections and failures.

### 2. Core concepts and vocabulary
- **Rollup / aggregate table** — pre-summarized data at a coarser grain (host × day).
- **Downsampling** — reducing resolution (per-second → per-minute) to save space; raw is purged by retention.
- **Watermark / high-water mark** — the position (timestamp, id, LSN) up to which data was processed.
- **Late-arriving data** — rows whose event time is older than the current watermark.
- **Full refresh** — truncate and rebuild.
- **CDC (change data capture)** — reading the source's change log (inserts/updates/deletes).
- **Idempotent** — re-running yields the same result.
- **Reconciliation** — comparing sums/counts between raw and aggregate.
- **Additive measure** — can be summed across buckets (count, sum); averages/percentiles are *not* additive (store sum and count; percentiles need sketches).

### 3. How it works, step by step
1. **Read the watermark** from a control table (`rollup_state(table, last_processed_ts)`).
2. **Define the slice**: `ts > watermark - overlap` and `ts <= new_watermark` where `new_watermark` is a *safe* point (e.g. now − 10 minutes) so in-flight writes are finished.
3. **Aggregate the slice** to the rollup grain (`GROUP BY host, bucket`).
4. **Merge idempotently**: either *replace* every affected bucket (delete those buckets then insert — recomputing each bucket from raw), or *add* deltas guarded by a batch id.
5. **Advance the watermark** in the *same transaction* as the merge.
6. **Reconcile**: check raw count per bucket equals rollup count.

Why `MAX(updated_at)` from the last run alone is unsafe: rows can be committed with an older timestamp *after* you read (long transactions, replication lag, clock skew), and rows with the same timestamp as the watermark can straddle two runs. Use an overlap window and idempotent bucket replacement, or a monotonic sequence/LSN from the source.

### 4. Worked example
```sql
-- rollup_state(name PK, watermark datetime2); metrics_raw(host, ts, cpu, mem); metrics_5m(host, bucket, n, sum_cpu, max_cpu, PK(host,bucket))
DECLARE @from datetime2, @to datetime2 = DATEADD(MINUTE, -10, SYSUTCDATETIME());   -- safe upper bound
SELECT @from = DATEADD(HOUR, -1, watermark)          -- overlap: reprocess last hour for late rows
FROM   dbo.rollup_state WHERE name = 'metrics_5m';

BEGIN TRAN;

-- Recompute every bucket touched by the slice from RAW (replace, not add) => idempotent
WITH slice AS (
    SELECT host,
           DATEADD(MINUTE, (DATEDIFF(MINUTE, 0, ts) / 5) * 5, 0) AS bucket,
           COUNT(*) AS n, SUM(cpu) AS sum_cpu, MAX(cpu) AS max_cpu
    FROM   dbo.metrics_raw
    WHERE  ts >= DATEADD(MINUTE, (DATEDIFF(MINUTE, 0, @from) / 5) * 5, 0)   -- align to bucket start
      AND  ts <  @to
    GROUP  BY host, DATEADD(MINUTE, (DATEDIFF(MINUTE, 0, ts) / 5) * 5, 0)
)
MERGE dbo.metrics_5m AS t
USING slice AS s ON t.host = s.host AND t.bucket = s.bucket
WHEN MATCHED THEN UPDATE SET t.n = s.n, t.sum_cpu = s.sum_cpu, t.max_cpu = s.max_cpu
WHEN NOT MATCHED THEN INSERT (host, bucket, n, sum_cpu, max_cpu) VALUES (s.host, s.bucket, s.n, s.sum_cpu, s.max_cpu);

UPDATE dbo.rollup_state SET watermark = @to WHERE name = 'metrics_5m';
COMMIT;

-- Reconciliation: raw vs rollup per bucket (should return no rows)
SELECT r.bucket, r.raw_n, a.n AS rollup_n
FROM  (SELECT DATEADD(MINUTE,(DATEDIFF(MINUTE,0,ts)/5)*5,0) AS bucket, COUNT(*) AS raw_n
       FROM dbo.metrics_raw WHERE ts < @to GROUP BY DATEADD(MINUTE,(DATEDIFF(MINUTE,0,ts)/5)*5,0)) r
JOIN  (SELECT bucket, SUM(n) AS n FROM dbo.metrics_5m GROUP BY bucket) a ON a.bucket = r.bucket
WHERE r.raw_n <> a.n;

-- Downsample to daily from the 5-minute rollup (never from raw again)
SELECT host, CAST(bucket AS date) AS d, SUM(n) AS n, SUM(sum_cpu) * 1.0 / SUM(n) AS avg_cpu, MAX(max_cpu) AS max_cpu
FROM   dbo.metrics_5m GROUP BY host, CAST(bucket AS date);
```
Because each touched bucket is **recomputed from raw and overwritten**, running the block twice yields identical results; a mid-run crash rolls back both merge and watermark. Note the daily average is `SUM(sum)/SUM(n)`, never `AVG(avg)`.

### 5. Under the hood (optional at L4)
A scan restricted by `ts >= @from AND ts < @to` is only cheap if `metrics_raw` is clustered or partitioned by time (or has an index on `ts`); otherwise the "incremental" step scans everything and defeats the point. Partitioning by day also lets old raw partitions be dropped by retention metadata-only. Full refresh vs CDC: a nightly full refresh re-reads everything but is trivially correct (including deletes and corrections); CDC reads the source's transaction log so it sees updates and deletes, but you must apply them (an update to an old event changes an old bucket — which is why *bucket recomputation from raw* handles corrections naturally while *delta addition* doesn't). If a pipeline's runtime jumps from 20 minutes to 4 hours after a schema change, isolate: compare plans (new scan/spill/implicit conversion), statistics (stale after the change), data volume, and blocking/wait stats; Query Store plan comparison pinpoints a regressed plan quickly.

### 6. Performance and design implications
- Make the slice predicate **sargable and aligned** to the clustered key/partition.
- Store *additive* components (count, sum, min, max, sum of squares) so higher roll-ups derive from the rollup, not raw.
- Percentiles can't be re-aggregated; store t-digest/HLL sketches or keep raw longer.
- Overlap window size = maximum expected lateness; beyond it, run a periodic backfill.
- Keep rollup grain coarse enough to be small, fine enough for the questions asked.
- Retention: keep raw N days, 5-minute rollup M months, daily forever.
- For ML feature tables, maintaining aggregates incrementally lets retraining read pre-summarized rows; corrections handled by recomputing affected buckets.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `MERGE` / delete+insert by bucket | Idempotent apply | Rollup upsert |
| Control/state table | Persist watermark | Every incremental job |
| Partition `SWITCH` / `DROP` | Fast retention | Raw expiry |
| Change Tracking / CDC (`sys.sp_cdc_enable_table`) | Read source changes | Updates/deletes upstream |
| `DATEDIFF`/`DATEADD` bucketing | Time buckets | 5m/1h/1d |
| Query Store | Compare plans over time | Regression triage |
| `UPDATE STATISTICS` | Refresh stats | After big loads/schema change |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Incremental rollup: large append-mostly data, tight window | Small tables — full refresh is simpler |
| Full refresh: small data, frequent source corrections | Data grows without bound |
| CDC: need deletes/updates reflected | Source can't expose a log or change tracking |
| Bucket replace | Buckets are huge and corrections rare — deltas may be cheaper |

### 9. Common mistakes
- Watermark = `MAX(updated_at)` with no overlap → late rows missed → overlap + recompute.
- Advancing the watermark outside the merge transaction → lost or duplicate slice → one transaction.
- Adding deltas without a batch guard → restart double-counts → replace buckets or track batch ids.
- Averaging averages → wrong mean → keep sum and count.
- Slice predicate on a non-indexed column → full scans → align with the cluster/partition key.
- Never reconciling → silent drift → count/sum check per bucket.
- Using local time for bucketing → DST duplicates/holes → UTC.

### 10. 🎯 Interview kit
**30-second answer:** "I keep a watermark and aggregate only data since it, with a small overlap for late rows, then idempotently replace the affected buckets and advance the watermark in the same transaction. I reconcile raw counts against the rollup, and store sums and counts so coarser rollups derive from it."

**Structure a longer answer:** clarify (grain, lateness, corrections, volume) → mechanism (watermark, slice, bucket replace) → trade-offs (full refresh vs incremental vs CDC) → scenario (restart mid-run doesn't double count).

**Likely follow-ups:**
- *What is a watermark?* → processed-up-to position used in `WHERE`.
- *Why is `MAX(updated_at)` alone unsafe?* → late/straddling rows and uncommitted transactions.
- *How do you verify losslessness?* → per-bucket raw count vs rollup count.
- *Job slowed from 20 min to 4 h?* → check plan, stats, volume, blocking via Query Store.

**Signals interviewers look for:** idempotency, late-data handling, additive-measure awareness.

### 11. Check yourself
- [ ] Write a watermark-driven rollup that is safe to re-run.
- [ ] Explain why averages must be stored as sum and count.
- [ ] Describe how late rows are handled with an overlap.
- [ ] Write the reconciliation query.
- [ ] Decide full refresh vs incremental for a 5 GB and a 5 TB table.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [What is a watermark column in incremental extraction, and how do you use it in a WHERE clause to pull only new data?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40005_What_is_a_watermark_column_in_incremental_extraction_and_how.md>)
- [Intermediate] [Your dbt incremental model uses a merge strategy but rebuilds full history every run. How would you diagnose why the incremental predicate isn't limiting the source scan and fix the is_incremental block?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40092_Your_dbt_incremental_model_uses_a_merge_strategy_but_rebuild.md>)
- [Deep] [What is the difference between a scheduled full refresh and a CDC-based incremental pipeline, and when would a fresher pick each?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40079_What_is_the_difference_between_a_scheduled_full_refresh_and_.md>)
- [Deep] [How would you compute a watermark/high-water-mark for incremental extraction that is safe against late-arriving source rows, and why is MAX(updated_at) from the last run alone unsafe?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40158_How_would_you_compute_a_watermark_high-water-mark_for_increm.md>)
- [Deep] [Your nightly feature-materialization job double-counts events on the day the pipeline restarted mid-run. How would you make the SQL materialization idempotent and safely re-runnable?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40612_Your_nightly_feature-materialization_job_double-counts_event.md>)
- [Deep] [Explain how you'd write a SQL check that verifies your telemetry rollup is lossless by reconciling the summed raw sample count against the rollup's total count per bucket, and what a mismatch tells you.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-41144_Explain_how_you_d_write_a_SQL_check_that_verifies_your_telem.md>)
- [Expert] [A nightly ETL job that used to finish in 20 minutes now runs for 4 hours after a schema change. Walk me through how you would isolate whether the regression is a bad plan, missing statistics, lock contention, or data-volume growth.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40278_A_nightly_ETL_job_that_used_to_finish_in_20_minutes_now_runs.md>)
- [Expert] [How would you design a SQL rollup that maintains daily feature aggregates incrementally so retraining reads pre-summarized rows instead of re-scanning raw events, and how do you handle corrections?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40704_How_would_you_design_a_SQL_rollup_that_maintains_daily_featu.md>)

## Related concepts

- [dbt Models and Runs](<../../../TOOLS/dbt/dbt_Core/dbt_Models_and_Runs.md>) *(dbt Core)*
- [Handling Source Changes and Spikes](<../../../02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Handling_Source_Changes_and_Spikes.md>) *(ETL ELT)*
- [Idempotent Incremental Loads](<../../../02_Data_Engineering/Pipelines_and_Integration/Idempotent_Incremental_Loads/Idempotent_Incremental_Loads.md>) *(Idempotent Incremental Loads)*

## References

- *Building Medallion Architectures* — Piethein Strengholt — Demystifying the Medallion Architecture › Processing Full Data Loads
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ingestion Design Patterns › Solution
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ingestion Design Patterns › Consequences
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ingestion Design Patterns › Examples
- *Data Engineering Design Patterns* — Bartosz Konieczny — Idempotency Design Patterns › Solution
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Best practices
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Data Cleanup
- *Data Pipelines Pocket Reference* — James Densmore — Transforming Data › Modeling Incrementally Ingested Data
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Incremental Statistics
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Automating Statistics Maintenance
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 15: Case Studies and the Future of T-SQL › Real-World Example

