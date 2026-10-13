---
id: "02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/CDC_Fundamentals_and_Ordering"
level: 4
type: "concept"
topic: "02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture"
tools: ["Apache_Kafka"]
---

# CDC Fundamentals and Ordering

**L4 Advanced** · [↑ Change Data Capture](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Change data capture turns a database's own record of what changed (inserts, updates, deletes) into an ordered stream you can replay elsewhere, instead of repeatedly copying whole tables.

### 1. The big picture
An operational database holds the *current state* of the business. Analytics, search indexes, caches and other services need that state too, and they need it fresh. The naive approach is to copy tables on a schedule. That is expensive (you move rows that did not change), slow (latency equals the schedule), and lossy (you only see the state at copy time, not what happened in between).

Change data capture (CDC) flips the question from "what does the table look like now?" to "what changed since I last looked?". Each change event (insert, update, delete) is extracted and delivered downstream, either in near real time or in small batches. It sits at the very start of the pipeline, in the *ingestion* stage, and feeds everything after it: landing zones, merge/upsert steps, streaming joins, search indexes.

The hard part is not reading changes; it is preserving **correctness**: no lost events, no duplicates applied twice, and events applied in an order that yields the same final state as the source. This lesson covers the three CDC styles, what a change event looks like, and why ordering is the central design problem.

### 2. Core concepts and vocabulary
- **Change event** — one record describing one row-level change: which table, which operation, the key, and (usually) the values.
- **Log-based CDC** — reading the database's own transaction log (MySQL binlog, PostgreSQL write-ahead log, SQL Server transaction log / CDC tables, Oracle redo). Lowest impact on the source because the log already exists for crash recovery and replication.
- **Query-based (batch/timestamp) CDC** — periodically querying the table for rows whose `updated_at` (or version number, or status flag) is newer than the last extraction.
- **Trigger-based CDC** — database triggers write a copy of each change into an audit/shadow table in the same transaction.
- **Before and after image** — the row values before and after the change. Updates carry both; inserts have only "after"; deletes only "before".
- **Operation type** — usually `insert`, `update`, `delete` (and sometimes `snapshot/read` for the initial copy, `truncate`).
- **Log position / offset (LSN, binlog file+position, GTID)** — a monotonically increasing pointer into the log. Your CDC reader stores it so it can resume exactly where it stopped.
- **Soft delete** — a delete represented as an updated column (`is_deleted = true`) instead of removing the row. Query-based CDC can see these; it cannot see physical deletes.
- **Transaction boundary** — the set of changes committed atomically together. Correct downstream reconstruction respects it.

### 3. How it works, step by step
**Query-based (timestamp) CDC**
1. The pipeline remembers a high-water mark, e.g. `last_extracted = 2026-10-11 23:00`.
2. It runs `SELECT ... WHERE updated_at > :last_extracted` against the source.
3. It merges the returned rows into the target by primary key.
4. It advances the high-water mark to the maximum `updated_at` it saw.

Key limitations:
- You see only the *latest version* of each changed row. If an account balance changed five times in 24 hours you get one row with the final balance, not five events. Intermediate states are lost.
- Physical deletes are invisible (the row is simply not returned).
- It needs a trustworthy `updated_at`; applications that bypass it, or batch jobs that update rows without touching it, create silent gaps.
- It adds query load to the production database, so you run it off-hours or against a read replica.

**Log-based CDC**
1. The database writes every committed change to its transaction log (it does this anyway for durability and replication).
2. A CDC reader attaches as a log consumer (a replication client). It parses log records, decodes them into row-level events, and groups them by transaction.
3. Events are emitted in **commit order** with their log position.
4. The reader persists its last-processed position. On restart it resumes from there.
5. A downstream consumer applies events to the target (merge, append, or publish to a stream).

Because the log records every change, you capture all intermediate states and physical deletes, and the source database does almost no extra work. The cost is operational: you must enable the right log level, keep log segments available while the reader is behind, and handle schema changes carefully.

**Trigger-based CDC**
1. Each source table gets `AFTER INSERT/UPDATE/DELETE` triggers.
2. Triggers write change rows into a `*_changes` table inside the same transaction.
3. A reader polls that table and deletes/marks consumed rows.

It captures everything and works where logs are unavailable, but it adds write latency and overhead to every transaction on the source, and someone must maintain triggers across schema changes.

| Style | Sees intermediate states | Sees deletes | Source impact | Needs source changes |
|---|---|---|---|---|
| Query-based | No | Only soft deletes | Query load | A reliable `updated_at` |
| Trigger-based | Yes | Yes | Added write cost | Triggers + shadow tables |
| Log-based | Yes | Yes | Very low | Log config / permissions |

### 4. Worked example
A `payments` table in PostgreSQL. With log-based CDC the following statements:

```sql
INSERT INTO payments (id, status, amount) VALUES (42, 'pending', 100);
UPDATE payments SET status = 'authorized' WHERE id = 42;
UPDATE payments SET status = 'captured', amount = 95 WHERE id = 42;
DELETE FROM payments WHERE id = 42;
```

produce four events in log order:

| lsn | op | before | after |
|---|---|---|---|
| 0/1A01 | insert | null | {id:42, status:pending, amount:100} |
| 0/1A02 | update | {…pending, 100} | {…authorized, 100} |
| 0/1A03 | update | {…authorized, 100} | {…captured, 95} |
| 0/1A04 | delete | {…captured, 95} | null |

A nightly timestamp-based extract running after all four statements would return **nothing** for id 42: the row is gone, and the earlier states were never seen. If instead the delete had not happened, it would return one row (`captured`, 95) and the pipeline would never know there had been a `pending` or `authorized` state, which may be exactly what an analyst needs (time-to-capture).

**Applying events to a warehouse table** (target keyed by `id`): for each event in order, `insert/update` becomes an upsert of the "after" image; `delete` removes the key or sets a deleted flag. If you instead append every event to a history table, you can later rebuild "current state" with a window function:

```sql
SELECT * FROM (
  SELECT *, ROW_NUMBER() OVER (PARTITION BY id ORDER BY lsn DESC) rn
  FROM payments_changes
) t WHERE rn = 1 AND op <> 'delete';
```

### 5. Under the hood
**Why ordering matters.** Applying "update to captured" before "update to authorized" leaves the target with the wrong final value. Ordering requirements:
- **Per key**, events must be applied in commit order. That is the minimum for correctness of the final state.
- **Across keys within a transaction**, you may need atomic application if downstream readers must never see half a transaction (a transfer that debits one account and credits another).

**Where ordering breaks.**
- *Partitioned transport.* If events go through a partitioned log (e.g. Kafka), ordering is only guaranteed within a partition. Use the primary key as the message key so all events for a row land in the same partition.
- *Parallel appliers.* Multiple writer threads can reorder events for the same key unless work is sharded by key.
- *Retries and replays.* At-least-once delivery means an event may arrive twice. Application must be **idempotent**: applying the same event twice gives the same result. Upserts by key plus a "only apply if this log position is greater than the stored one" guard achieve this.
- *Clock-based ordering.* Using wall-clock timestamps to order events is fragile because of clock skew between servers; prefer the log position, which the database assigns.
- *Snapshot + stream overlap.* When you first load a table (snapshot) while the log keeps moving, the snapshot and the first streamed events overlap. The reader must record the log position at snapshot start and replay from there, or you lose or duplicate changes.

**Log retention.** Logs are circular/recycled. If the CDC reader falls behind further than the retention window, the position it needs no longer exists and you must re-snapshot. Conversely, a stalled reader (for example a PostgreSQL replication slot that is not consumed) can force the database to keep WAL forever and fill the disk.

**Schema change.** Events are decoded using the table definition at the time of the change. Adding or dropping columns mid-stream needs a defined policy: propagate the schema change, tolerate extra fields, or stop and alert.

### 6. Performance and design implications
- CDC is not free: it consumes memory, disk bandwidth, storage, CPU and network on the source. Load-test on production-like volume before enabling it, as you would with synchronous replication.
- Batch CDC against a primary can create heavy query load; use off-hours windows or a read replica.
- Large transactions (bulk updates of millions of rows) produce a burst of events; plan buffering and consumer capacity.
- Measure **end-to-end lag**: source commit time to target visibility. Also track log-reader lag in bytes or seconds, and the age of the oldest unconsumed log position.
- Keep a **reconciliation** job (row counts, checksums on key ranges) as a safety net; CDC pipelines can drift silently.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| MySQL `binlog_format=ROW`, `binlog_row_image=FULL` | Logs row-level before/after images | Required for log-based CDC on MySQL |
| PostgreSQL `wal_level=logical` | Enables logical decoding | Required for log-based CDC on PostgreSQL |
| `pg_create_logical_replication_slot` / `pg_replication_slots` | Create / inspect slots; shows lag and retained WAL | Monitor to avoid disk fill |
| SQL Server `sys.sp_cdc_enable_table` | Enables built-in CDC on a table | When you want change tables inside SQL Server |
| `updated_at` + index on it | Supports query-based CDC | When logs are unavailable; index it or the extract scans the table |
| Replica identity (PG `REPLICA IDENTITY FULL`) | Includes full old row in updates/deletes | When you need the before image of tables without a suitable key |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Log-based CDC: you need every change, deletes, low source impact, low latency | You cannot get log access (some managed or legacy systems), or the team cannot operate the plumbing |
| Query-based: simple tables with a trusted `updated_at`, hourly/daily freshness is fine | You need deletes or intermediate states, or `updated_at` is unreliable |
| Trigger-based: no log access, small write volume, need full history | Write-heavy tables where trigger overhead hurts |
| Batch CDC (hourly) when business freshness tolerance is loose and cost matters | Use cases like fraud detection where minutes matter |

### 9. Common mistakes
- Trusting `updated_at` → rows modified by bulk jobs or direct SQL never appear → add a log-based path or periodic reconciliation.
- Ignoring physical deletes → the target keeps "ghost" rows forever → use log-based CDC or soft deletes, or run a key-set comparison.
- Ordering by wall-clock time or arrival time → wrong final state → order by log position per key.
- Not keying the stream by primary key → same-row events land on different partitions and race → key by primary key.
- Forgetting idempotency → retries double-count in aggregates → upsert by key with a position guard.
- Enabling CDC without capacity testing → source slows down or runs out of disk → test, monitor retained log, set alerts.

### 10. 🎯 Interview kit
**30-second answer:** CDC captures row-level inserts, updates and deletes from a source database and ships them downstream so you keep a copy in sync without full reloads. The best form is log-based because it reads the transaction log the database already writes, giving complete, ordered, low-impact changes. The key challenges are ordering per key, idempotent application, and handling snapshots and schema changes.

**Structure a longer answer:** clarify freshness and source type → name the three styles and what each can/cannot see → explain log-based mechanics and positions → ordering and idempotency → operational risks (retention, slots, schema drift) → close with a real scenario.

**Likely follow-ups:**
- *What is in a CDC event?* → operation type, before/after images, key, source metadata (table, log position, commit time, transaction id).
- *How do you keep order?* → per-key ordering by log position; partition by primary key; single writer per key.
- *How do you resync a drifted target?* → incremental or full snapshot with a recorded log position, then resume the stream; verify with counts/checksums.
- *What is a delete storm and how do you guard against it?* → a bad job deletes many rows and CDC faithfully replicates it; add thresholds that pause application and alert, and keep history so you can restore.
- *Transactional consistency downstream?* → use transaction metadata to apply all events of a commit together, or accept per-row eventual consistency.

**Signals interviewers look for:** you distinguish log/query/trigger CDC with concrete limits, you talk about ordering and idempotency unprompted, and you mention operations (lag, retention, reconciliation).

### 11. Check yourself
- [ ] Explain why timestamp-based CDC loses intermediate updates and physical deletes.
- [ ] Draw the path of one update from a database commit to a warehouse row, naming where the log position is stored.
- [ ] State two ways events for the same key can be applied out of order and the fix for each.
- [ ] Describe how to combine an initial snapshot with a streaming log without gaps or duplicates.
- [ ] List three signals you would monitor on a log-based CDC pipeline.
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)

## Practice questions

- [Basic] [What is Change Data Capture (CDC) and why is it useful in a data pipeline?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/Q-06009_What_is_Change_Data_Capture_CDC_and_why_is_it_useful_in_a_da.md>)
- [Intermediate] [What information does a typical CDC event payload contain (before after and metadata)?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/Q-06025_What_information_does_a_typical_CDC_event_payload_contain_be.md>)
- [Intermediate] [How would you validate that the CDC target row count matches the source after a load?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/Q-06069_How_would_you_validate_that_the_CDC_target_row_count_matches.md>)
- [Deep] [What does transactional consistency mean when reconstructing data from CDC events?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/Q-06008_What_does_transactional_consistency_mean_when_reconstructing.md>)
- [Deep] [If your warehouse table drifts out of sync with the source how would you resync it with CDC?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/Q-06089_If_your_warehouse_table_drifts_out_of_sync_with_the_source_h.md>)
- [Deep] [How do you design CDC ingestion so a downstream backfill and the live stream don't clobber each other's rows during the overlap window?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/Q-06128_How_do_you_design_CDC_ingestion_so_a_downstream_backfill_and.md>)
- [Deep] [A CDC delete storm from a bad batch job at the source is propagating to the warehouse: how would you detect it and add a safety valve to stop cascade damage?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/Q-06165_A_CDC_delete_storm_from_a_bad_batch_job_at_the_source_is_pro.md>)
- [Deep] [How do you handle CDC for temporal or bitemporal source tables where the source itself already tracks valid-time versions?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/Q-06195_How_do_you_handle_CDC_for_temporal_or_bitemporal_source_tabl.md>)

## Related concepts

- [Topic and Schema Design for Pipelines](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Topic_and_Schema_Design_for_Pipelines.md>) *(Big Data Processing)*
- [Pipeline Design Patterns](<../Pipeline_Design/Pipeline_Design_Patterns.md>) *(Pipeline Design)*
- [End to End No Data Loss](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/End_to_End_No_Data_Loss.md>) *(Data Architecture)*
- [Data Warehouse Fundamentals](<../../Modeling_and_Warehousing/Data_Warehousing/Data_Warehouse_Fundamentals.md>) *(Data Warehousing)*
- [Keeping Indexes in Sync via CDC](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/Keeping_Indexes_in_Sync_via_CDC.md>) *(Distributed Systems Core)*
- [Handling Source Changes and Spikes](<../ETL_ELT/Handling_Source_Changes_and_Spikes.md>) *(ETL ELT)*

## References

- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Generation in Source Systems › Change Data Capture
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Change Data Capture
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Continuous CDC
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Queries on Streaming Data › The Kappa architecture
- *Building Data Integration Solutions* — Jay Borthen — Key Concepts in Data Integration › Change Data Capture
- *Building Medallion Architectures* — Piethein Strengholt — Laying the Groundwork › Change Data Capture
- *Building Medallion Architectures* — Piethein Strengholt — Construct the Bronze Layer › Using Change Data Capture
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Capturing Data Changes
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Capturing Data Changes › CDC Feed
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Capturing Data Changes › CDC Sources
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Capturing Data Changes › CDC agents
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Change Data Capture › Chapter 13. Change Data Capture
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ingestion Design Patterns › Pattern: Change Data Capture
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ingestion Design Patterns › Consequences
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ingestion Design Patterns › Payload
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ingestion Design Patterns › Examples

