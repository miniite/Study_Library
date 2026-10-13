---
id: "02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Concurrent_Partition_Overwrites"
level: 4
type: "concept"
topic: "02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing"
tools: ["Apache_Spark", "Hadoop"]
---

# Concurrent Partition Overwrites

**L4 Advanced** · [↑ Partitioning and Bucketing](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Overwriting a partition is only safe when the same partition is never rewritten by two jobs at once and each overwrite is atomic and idempotent; concurrency control, partition-scoped replacement and per-partition validation are what make backfills and late-data fixes trustworthy.

### 1. The big picture
Partitioned tables are the unit of reprocessing in batch data engineering: a daily job writes `event_date=2026-10-01`, and when something is wrong — late data, a bug fix, a backfill — you recompute that partition and replace it. This is the **overwrite** pattern, and it is attractive because replacing a whole partition is naturally idempotent: run it twice, get the same result.

The risks appear when more than one writer touches the same table. A backfill job and the regular daily job both overwrite the same date; two late-data runs both fix 2024-12-10; an orchestrator retries a task while the original is still running. Depending on the storage layer you can get lost updates, duplicated rows, or half-written partitions. The books illustrate this with the *Dynamic Late Data Integrator* pattern: each run discovers which partitions received late data and reprocesses them, saving resources compared to a fixed lookback window — but with concurrent executions the same partition can be picked up by several runs, so you must track which runs own which partitions.

A related theme is *not rewriting whole partitions when you only need part of them*: overwriting only the impacted entities saves compute but is harder to implement and to make safe.

### 2. Core concepts and vocabulary
- **Partition overwrite** — replace all data of a partition with newly computed data.
- **Static vs dynamic overwrite** — static names the partition explicitly; dynamic replaces whichever partitions appear in the written data.
- **Idempotent write** — repeating the operation leaves the same final state.
- **Atomic commit** — the new partition content becomes visible all at once.
- **Optimistic concurrency control (OCC)** — writers proceed without locks and validate at commit time that nothing conflicting changed; a conflict fails the commit.
- **Write skew / lost update** — two writers each based on an old read, one overwrites the other's result.
- **Partition lock / lease** — a marker (in a control table, orchestrator pool, or lock service) to serialise work on a partition.
- **Late data** — records arriving after the partition was first processed.
- **Run ID / batch ID column** — tag of which execution produced a row; helps detect duplicates and rollback.
- **Watermark / high-water mark** — how far processing has progressed.

### 3. How it works, step by step
**Safe partition replacement** (generic recipe):
1. Compute the new partition content into a staging location or a transient dataset — never directly in place.
2. Validate it (row count within tolerance, non-null keys, uniqueness, date column equals partition value).
3. Commit atomically: table-format commit that removes old files and adds new ones for that partition (Delta `replaceWhere`, Iceberg `INSERT OVERWRITE` with dynamic mode or `MERGE`, partition-level `ALTER TABLE ... EXCHANGE PARTITION` in warehouses).
4. Record the commit with run ID and row counts in a control/audit table.

**Handling concurrent writers:**
1. Prefer a table format with ACID commits so two commits cannot interleave file lists.
2. Make jobs for *different* partitions disjoint; table formats can then commit both because their conflict detection is partition-aware (Delta: predicates on partition columns; Iceberg: overwrite filters).
3. Serialise jobs targeting the *same* partition: orchestrator concurrency limits per partition key, a lock row, or a single "late data integrator" that owns all repairs.
4. On a conflict, retry from step 1 (re-read, recompute) — do not blindly retry the commit with stale files.

**Late-data integration with concurrency:** keep a table of partitions-to-reprocess (`partition`, `status`, `owner_run_id`). A run claims partitions atomically (`UPDATE ... WHERE status='pending'`); other runs skip claimed partitions. After success it marks them done. This avoids the duplicate executions the book shows when four parallel runs all see late data in the same two partitions.

### 4. Worked example
```sql
-- Delta: replace exactly one day, atomically; fails (and can be retried) on conflict
INSERT INTO silver.events
REPLACE WHERE event_date = DATE'2026-10-01'
SELECT * FROM staging.events_20261001_recomputed;

-- Iceberg (Spark): dynamic overwrite touches only partitions present in the source
SET spark.sql.sources.partitionOverwriteMode = dynamic;
INSERT OVERWRITE silver.events
SELECT * FROM staging.late_corrected;      -- contains only impacted dates

-- Pre-commit validation: every row belongs to the partition being replaced
SELECT count(*) AS bad_rows
FROM   staging.events_20261001_recomputed
WHERE  event_date <> DATE'2026-10-01';   -- must be 0

-- Control table: claim a partition for repair
UPDATE ops.partition_repairs
SET    status = 'running', owner_run_id = :run_id
WHERE  partition_key = '2026-10-01' AND status = 'pending';
-- check rows_affected = 1 before starting; otherwise another run owns it
```
What to look at: the table history (`DESCRIBE HISTORY` / `snapshots`) should show one commit per partition replacement with the expected row counts; a spike in failed commits indicates two writers fighting over one partition.

### 5. Under the hood
- **Why object stores need a table format:** "overwrite" on raw files means delete then write; readers can see an empty or partial partition, and a crash leaves the partition missing. A table format makes the swap a single metadata commit.
- **Conflict rules (Delta):** a write conflicts if a concurrent commit added/removed files that match its read predicate. Appends to different partitions are compatible; two overwrites of the same partition are not. Isolation level (`WriteSerializable` vs `Serializable`) changes how strict the check is.
- **Dynamic partition overwrite pitfall:** if the input accidentally contains an unexpected partition value (bad timestamp parsing producing 1970-01-01), dynamic overwrite silently replaces that partition too. Validate that distinct partition values match the intended set before committing.
- **Partial entity overwrite:** to fix only impacted customers, use `MERGE` or delete-then-insert by key inside one transaction; plain file-level overwrite cannot do this.
- **Stream + batch writers:** a streaming append and a batch overwrite on the same partition can both succeed but yield duplicates unless keys/dedup logic exist; avoid mixing, or have the batch correct via MERGE on keys.
- **Orchestrator side:** one task per partition scales badly (thousands of tasks hurt the scheduler); use dynamic task mapping or batch partitions into chunks, and give each chunk a concurrency key.

### 6. Performance and design implications
- Whole-partition overwrite costs compute proportional to partition size; entity-level fixes cost less but need keys and MERGE.
- Validation should be scoped to changed partitions only — running a full-table suite for every run becomes the slowest step in the DAG.
- Retry storms on OCC conflicts waste compute; stagger or serialise heavy writers.
- File sizes: repeatedly overwriting small partitions can create small files; compact afterwards.
- Choose event-time partitioning for correctness of analytics; use processing-time partitions when you need a simple, append-only landing zone with out-of-order sources and fix up downstream.
- Metrics: commit conflict count, retry count, partitions rewritten per run, time to repair a late-data incident.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `INSERT ... REPLACE WHERE` (Delta) | Atomic predicate-scoped overwrite | Replace specific partitions |
| `spark.sql.sources.partitionOverwriteMode=dynamic` | Overwrite only partitions present in data | Late-data repairs |
| `MERGE INTO` | Key-level upsert/delete | Entity-level correction |
| Isolation level (`delta.isolationLevel`) | Conflict strictness | Tuning concurrent writers |
| Orchestrator pools / `max_active_runs` | Limit parallelism | Serialise same-partition work |
| `DESCRIBE HISTORY` / `snapshots` metadata table | Inspect commits | Debug conflicts |
| Control/audit table | Claim partitions, record runs | Idempotency and lineage |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Whole-partition overwrite | Partition is huge and only a few keys changed |
| Key-level MERGE | No reliable business key or very heavy updates |
| OCC with retries | Many writers always target same partition — use locks/serialisation |
| Dynamic overwrite | Input may contain unexpected partition values |
| Fixed lookback window | Late data is rare and unpredictable — dynamic integrator is cheaper |

### 9. Common mistakes
- Overwriting in place without staging → partial partition visible → stage then commit atomically.
- Retrying a failed commit with stale file lists → lost updates → recompute from the latest snapshot.
- Leaving dynamic overwrite on with unvalidated partition values → wiping wrong partition → assert distinct partition set.
- Letting several late-data runs reprocess the same partition → duplicated work or double counts → claim partitions via a control table.
- Validating the whole table on every run → slow DAG → scope to changed partitions.
- Not checking that data landed in the correct partition → wrong date partitions → automated check that partition column equals expected value.
- Treating a missing day as success → silent gaps → explicit "no data" handling (empty partition marker or alert).

### 10. 🎯 Interview kit
**30-second answer:** Partition overwrite is idempotent only if the replacement is atomic and no other writer targets the same partition concurrently. I stage and validate the new data, commit through the table format with a predicate-scoped replace, serialise same-partition jobs with a control table or orchestrator limits, and rely on optimistic concurrency conflicts to catch races.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.

**Likely follow-ups:**
- *How do you validate landing in the correct partition?* → compare partition column to expected, row count vs source, distinct partition set, null partition keys.
- *No data arrived for a day?* → differentiate "no events" from "pipeline failure"; write an empty marker or fail and alert.
- *Late event three days old?* → MERGE or replace that partition, idempotently, tracked in a repair table.
- *Event-time vs processing-time partitioning for CDC?* → processing-time for landing (append-only), event-time in curated layers with key-based MERGE using source version.
- *Hourly sub-partition slowing writes?* → measure file count and size; likely over-partitioned; drop to daily and cluster on hour.

**Signals interviewers look for:** atomicity, idempotency, concurrency control and scoped validation, not just "use overwrite".

### 11. Check yourself
- [ ] Explain why overwriting files directly on object storage is unsafe for readers.
- [ ] Describe what happens when two jobs overwrite different partitions of one Delta table, and when they overwrite the same one.
- [ ] Design a control table to prevent duplicate late-data runs.
- [ ] Write a pre-commit validation that guards dynamic partition overwrite.
<!-- explainer:end -->

## Implementations (tools)

- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)
- [Hadoop](<../../../TOOLS/Hadoop/_index.md>)

## Practice questions

- [Intermediate] [How do you validate that data landed in the correct partition after an ETL load, and what checks do you automate?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Q-35367_How_do_you_validate_that_data_landed_in_the_correct_partitio.md>)
- [Intermediate] [In an orchestrated pipeline, how would you handle a day where no data arrived for a partition?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Q-35488_In_an_orchestrated_pipeline_how_would_you_handle_a_day_where.md>)
- [Intermediate] [A Great Expectations suite runs on every partition and is now the slowest step in the DAG — how do you scope validation to changed partitions?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Q-35557_A_Great_Expectations_suite_runs_on_every_partition_and_is_no.md>)
- [Intermediate] [How do you decide whether repartitioning before a wide write is worth the extra shuffle it introduces to fix output file sizes?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Q-35582_How_do_you_decide_whether_repartitioning_before_a_wide_write.md>)
- [Deep] [You partitioned a Delta table by event_date and hour and now writes take 3x longer than reads warrant — how do you reason about whether the hour sub-partition is worth it?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Q-35497_You_partitioned_a_Delta_table_by_event_date_and_hour_and_now.md>)
- [Deep] [An Airflow DAG creates one task per partition and now has 3000 tasks per run — how do you redesign partition-aware orchestration to keep the scheduler healthy?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Q-35520_An_Airflow_DAG_creates_one_task_per_partition_and_now_has_30.md>)
- [Deep] [How would you detect and alert on partition skew in production before it turns into an on-call incident during a big traffic day?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Q-35558_How_would_you_detect_and_alert_on_partition_skew_in_producti.md>)
- [Deep] [Explain how you'd choose between event-time and processing-time partitioning for a CDC stream where source updates can arrive out of order.](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Q-35589_Explain_how_you_d_choose_between_event-time_and_processing-t.md>)

## Related concepts

- [Partitioning Strategy](<../../../01_Databases_and_SQL/Database_Operations/Partitioning/Partition_Pruning/Partitioning_Strategy.md>) *(Partitioning)*
- [End to End No Data Loss](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/End_to_End_No_Data_Loss.md>) *(Data Architecture)*

## References

- *Data Engineering Design Patterns* — Bartosz Konieczny — Error Management Design Patterns › Solution
- *Data Engineering Design Patterns* — Bartosz Konieczny — Error Management Design Patterns › Not Whole Partitions
- *Data Engineering Design Patterns* — Bartosz Konieczny — Error Management Design Patterns › Scheduling complexity
- *Data Engineering Design Patterns* — Bartosz Konieczny — Sessionization › Solution
- *Mapping Data Flows in Azure Data Factory* — Mark Kromer — Common ETL Pipeline Practices in ADF with Mapping Data Flows › Task 3: Evaluate Execution Plan
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — This is a simple example using hardcoded values for readability rather than variables. You can find
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — slightly different. A higher consistency level means that more nodes need to respond to the query
- *Hadoop: The Definitive Guide* — Tom White — Flume › Partitioning and Interceptors
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Index Maintenance › Columnstore Index Maintenance by Partition
- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioning › Operations: Automatic or Manual Rebalancing

