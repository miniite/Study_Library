---
id: "02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Handling_Source_Changes_and_Spikes"
level: 4
type: "concept"
topic: "02_Data_Engineering/Pipelines_and_Integration/ETL_ELT"
tools: []
---

# Handling Source Changes and Spikes

**L4 Advanced** · [↑ ETL ELT](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Sources change shape and volume without asking; a resilient pipeline detects schema drift and volume spikes early, absorbs them safely, and lets you re-run or backfill without damage.

### 1. The big picture
Pipelines break less because of bugs in your code than because the world moves: a source adds, renames or retypes a column; a file arrives with a different delimiter; an upstream team ships a feature that doubles event volume; a partner API starts rate limiting; data arrives late or out of order; time zones shift during daylight saving. The aim of this lesson is a design stance: assume change, detect it fast, contain its blast radius, and recover by re-running.

### 2. Core concepts and vocabulary
- **Schema drift** — unplanned change in the structure or types of source data.
- **Schema evolution** — managed handling of those changes (adding columns, widening types).
- **Backward-compatible change** — one existing readers can tolerate (adding an optional column).
- **Volume spike / skew** — sudden growth in total records or in records per key/partition.
- **Back-pressure** — slowing producers or buffering when consumers can't keep up.
- **Autoscaling** — adding resources automatically as load grows.
- **Late-arriving data** — records arriving after the window that logically contains them.
- **Backfill** — recomputing historical partitions.
- **Dead-letter / quarantine** — holding place for records that cannot be processed.
- **External trigger** — start processing on an event (file arrived, message published) instead of a clock.
- **Watermark** — a marker of how far event time has progressed, used to decide when a window is complete.

### 3. How it works, step by step
**Handling schema change**
1. **Detect:** compare incoming schema (file header, API response, table metadata) with the expected contract at the start of each run.
2. **Classify:** additive (new column), widening (int → bigint), narrowing/removal/rename/semantic change.
3. **Act by policy:** allow additive in raw, alert; block narrowing/removal in curated; require approval for semantic changes.
4. **Record:** log the change, version the schema, notify the owner.
5. **Re-run** affected models after adapting them.

**Handling volume spikes**
1. **Detect:** compare today's row counts and bytes to a baseline (same weekday, trailing average) and monitor job duration and lag.
2. **Absorb:** buffer in a queue or object storage; scale workers; raise batch/parallelism limits within safe bounds.
3. **Protect downstream:** rate-limit sinks; apply back-pressure; avoid retry storms.
4. **Investigate:** is the spike real (campaign), duplicated (replay, retry bug) or a runaway (loop in the producer)?
5. **Adjust:** change capacity, partitioning, or move from fixed schedule to event-driven triggers.

**Handling late data**
1. Partition by event time, not arrival time, where analytics requires it.
2. Keep a lookback window (reprocess last N days each run) or use a watermark with allowed lateness.
3. Upsert into the target by key so reprocessing corrects earlier output.

### 4. Worked example
A daily pipeline loads `clicks_2026-10-12.csv`. Today the file has a new column `device_type` in position 3 and a changed timestamp format.

```python
EXPECTED = ["click_id", "user_id", "ts", "url"]
cols = read_header(path)
added   = [c for c in cols if c not in EXPECTED]
missing = [c for c in EXPECTED if c not in cols]
if missing:                 # breaking: stop and alert
    raise SchemaBreak(missing)
if added:                   # additive: continue, record, notify
    log_schema_change(added)
df = read_csv(path, usecols=EXPECTED, parse_dates=["ts"])   # select by name, not position
```

Key habits shown: read by column *name*, tolerate additions, fail loudly on removals, and parse timestamps explicitly with a timezone.

Time zones and DST: an incremental load partitioned by local event date has a 23-hour and a 25-hour day each year. Store and partition in UTC; convert for presentation. Otherwise a "daily" window can skip or duplicate an hour.

Event-driven trigger: when the backend team publishes a feature, they emit a notification to a message bus; the pipeline subscribes and runs only then, instead of running every hour to find nothing. A low-frequency scheduled run remains as a safety net for lost notifications.

### 5. Under the hood
- **Schema-on-read vs write.** Landing raw as files/variant and parsing at read time absorbs additive drift without failing; strict typed tables fail on any mismatch. Use both: permissive raw, strict curated.
- **Positional vs named parsing.** Positional CSV parsing silently misaligns data when a column is inserted. Always bind by name or validate headers.
- **Type drift.** A numeric column starting to include strings fails casts. Use safe-cast with quarantine for failures and measure the failure rate.
- **Spikes vs skew.** Total volume doubling is a capacity problem; one hot key (a bot user, a null key) is a skew problem causing a single partition/task to dominate runtime. Fixes differ: scale vs salting/repartitioning/filtering.
- **Retries amplifying load.** A failing downstream plus aggressive retries creates a thundering herd. Use exponential backoff with jitter, circuit breakers and bounded queues.
- **Idempotency makes everything else safe.** If re-running a window produces the same output, you can respond to any change by fixing logic and re-running, without hand-cleaning.

### 6. Performance and design implications
- Build baselines: row count, byte size, null rate, distinct keys, duration per step; alert on deviations beyond a threshold, with seasonality (weekday) awareness.
- Keep headroom: concurrency limits and warehouse sizes that can handle 2–3× normal load for the batch window.
- Separate fast paths for critical tables from heavy, long-running ones so a spike in one does not starve others.
- Keep raw inputs immutable and retained so reprocessing is always possible.
- Provide a documented schema-change workflow with owners; contracts with producers reduce surprise (see data contracts).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| Schema evolution options (e.g. `mergeSchema`, `ALTER TABLE ... ADD COLUMN`) | Allow additive changes | Raw/bronze layers |
| `TRY_CAST` / safe-cast + quarantine | Avoid hard failure on bad values | Type drift |
| Partition by event date in UTC | Stable windows | Incremental/backfill |
| Queue/stream buffer (Kafka, SQS, Pub/Sub) | Absorb bursts | Spiky producers |
| Autoscaling / concurrency limits | Match capacity to load | Compute clusters, warehouses |
| Backoff + jitter retry policy | Prevent retry storms | API/DB calls |
| Anomaly checks (row count vs baseline) | Catch spikes/drops | After each load |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Permissive raw + strict curated | Teams read raw directly as if validated |
| Fail-fast on removals/type narrowing | Pipelines feeding non-critical, high-tolerance reports (use soft-fail) |
| Autoscale for unpredictable bursts | Cost-sensitive workloads where a queue and delay is acceptable |
| Event-driven triggers | Reliable cron suffices; added moving parts |
| Lookback reprocessing for late data | Huge tables where recomputation is costly (use watermark + merge) |

### 9. Common mistakes
- Parsing by column position → misaligned data silently → bind by name and validate.
- Auto-propagating all drift into curated models → downstream breakage → gate curated with contracts.
- Volume alerts with static thresholds → noise or misses → weekday-aware baselines.
- Retrying instantly → worse outage → backoff and jitter.
- Partitioning by local time → DST bugs → UTC.
- Assuming a spike is real without checking duplicates → inflated metrics → compare distinct keys and sources.

### 10. 🎯 Interview kit
**30-second answer:** I assume sources change. I validate schema at the boundary by name, allow additive changes in raw with alerts, block breaking ones in curated, and quarantine bad records. For volume spikes I buffer, scale, and rate-limit, and I investigate whether it is real or duplicate. Idempotent, partitioned loads let me backfill safely.

**Structure a longer answer:** detect → classify → contain → recover → prevent (contracts, monitoring).

**Likely follow-ups:**
- *Schema drift breaking a job?* → name-based parsing, evolution policy, contracts, alerts.
- *Source volume doubles?* → capacity, back-pressure, skew vs total, event-driven triggers.
- *Late data?* → event-time partitions, lookback/watermark, upsert.
- *DST edge cases?* → UTC storage and windows.
- *Flaky partner API?* → timeouts, backoff, idempotency keys, circuit breaker.

**Signals interviewers look for:** layered defenses and concrete mechanisms rather than "add monitoring".

### 11. Check yourself
- [ ] Write a policy table mapping schema changes to actions.
- [ ] Explain skew vs spike and a fix for each.
- [ ] Design a lookback strategy for events arriving up to 3 days late.
- [ ] Describe how idempotency simplifies recovery from drift.
- [ ] Show how to make a CSV parser resilient to inserted columns.
<!-- explainer:end -->

## Practice questions

- [Basic] [How would you remove duplicate rows during a transformation step in SQL?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Q-24767_How_would_you_remove_duplicate_rows_during_a_transformation_.md>)
- [Intermediate] [How would you handle a CSV file where some rows have extra or missing columns?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Q-24795_How_would_you_handle_a_CSV_file_where_some_rows_have_extra_o.md>)
- [Intermediate] [Explain how you'd detect and handle a source that starts sending truncated or partial files intermittently.](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Q-24919_Explain_how_you_d_detect_and_handle_a_source_that_starts_sen.md>)
- [Deep] [What is a checkpoint in a data pipeline and how does it aid recovery after a failure?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Q-24833_What_is_a_checkpoint_in_a_data_pipeline_and_how_does_it_aid_.md>)
- [Deep] [Your CDC pipeline from Debezium started lagging by hours during a bulk update on the source; how do you diagnose whether the bottleneck is Kafka, the connector, or the sink?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Q-24855_Your_CDC_pipeline_from_Debezium_started_lagging_by_hours_dur.md>)
- [Deep] [Explain how you'd handle hard deletes on the source when your CDC stream only reliably captures inserts and updates.](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Q-24873_Explain_how_you_d_handle_hard_deletes_on_the_source_when_you.md>)
- [Deep] [How do you approach loading semi-structured API responses whose schema evolves weekly, keeping historical queries stable?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Q-24905_How_do_you_approach_loading_semi-structured_API_responses_wh.md>)
- [Deep] [How do you handle a source primary key that isn't actually unique, and only discover it after months of loads?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Q-24924_How_do_you_handle_a_source_primary_key_that_isn_t_actually_u.md>)

## Related concepts

- [Data Warehouse Fundamentals](<../../Modeling_and_Warehousing/Data_Warehousing/Data_Warehouse_Fundamentals.md>) *(Data Warehousing)*
- [Idempotent Incremental Loads](<../Idempotent_Incremental_Loads/Idempotent_Incremental_Loads.md>) *(Idempotent Incremental Loads)*
- [End to End No Data Loss](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/End_to_End_No_Data_Loss.md>) *(Data Architecture)*
- [Incremental Rollups](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Incremental_Rollups.md>) *(SQL Fundamentals)*

## References

- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Backups › 12.5 #82 Taking backups during an ETL window
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Five Best Practices for Stable Data Processing What to Keep in Mind When Setting Up Processes Like › Consider Dependency on Other Systems
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — The End of ETL as We Know It › Replacing ETL with Intentional Data Transfer
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — The End of ETL as We Know It › Agreeing on a Data Model Contract
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 11: Automation with SSIS, SQL Agent, and PowerShell › Checkpoints
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 11: Automation with SSIS, SQL Agent, and PowerShell › Case Study: Automated ETL Error Handling
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 11: Automation with SSIS, SQL Agent, and PowerShell › Integrating Error Handling with DevOps
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 11: Automation with SSIS, SQL Agent, and PowerShell › Case Study: Enterprise ETL Error Handling
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ingestion Design Patterns › Incremental Load
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ingestion Design Patterns › Problem
- *Data Engineering Design Patterns* — Bartosz Konieczny — Error Management Design Patterns › Overlapping executions and backfilling
- *Data Engineering Design Patterns* — Bartosz Konieczny — Idempotency Design Patterns › Backfilling for Data Provider's Mistakes
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Continuous Integration and Deployment › Deployment
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Infrastructure Operations › Understanding ETL performance fundamentals
- *SQL Server Integration Services Design Patterns* — Tim Mitchell et al. — Read the project file, and deploy it to the folder byte $projectFile = System.IO.File::ReadAllBytes($ProjectFilePath) $project = $folder.DeployProject($ProjectName, $projectFile) › Building a Basic Incremental Load SSIS Package
- *SQL Server Integration Services Design Patterns* — Tim Mitchell et al. — Read the project file, and deploy it to the folder byte $projectFile = System.IO.File::ReadAllBytes($ProjectFilePath) $project = $folder.DeployProject($ProjectName, $projectFile) › Specifying a Data Flow Task

