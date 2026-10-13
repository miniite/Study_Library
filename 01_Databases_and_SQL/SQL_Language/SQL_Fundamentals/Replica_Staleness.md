---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Replica_Staleness"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: []
---

# Replica Staleness

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A read replica is a copy that applies the primary's changes a little later, so a read that goes to it can return data that is already out of date; you manage this by measuring the lag and by routing the few reads that must be fresh to the primary (or making them wait for the replica to catch up).

### 1. The big picture
Almost every production database grows a **read replica**: a second server that receives the primary's write-ahead log (WAL / binlog / transaction log) and replays it. Replicas let you spread read traffic, run reports without disturbing writers, and survive a primary failure. The price is **replication lag**: the time between a commit on the primary and the moment that commit becomes visible on the replica.

When replication is *asynchronous* (the default almost everywhere) the primary acknowledges a commit as soon as its own log is durable; it does not wait for replicas. So there is always a window, usually milliseconds, sometimes tens of seconds under load, in which a replica serves an older version of the data. Users experience this as "I saved my profile and the page shows the old one" or "my order vanished after checkout".

The same idea shows up elsewhere in the stack (caches, search indexes, warehouse copies fed by CDC, materialized views): a *derived copy* that is eventually consistent with its source. The vocabulary and the fixes below transfer directly. The parent topic (SQL fundamentals) matters here because the *queries* you write decide which consistency you need: a dashboard count can be stale by a minute, a "show me the row I just inserted" cannot.

### 2. Core concepts and vocabulary
- **Primary / source / leader** — the node that accepts writes.
- **Replica / standby / follower / secondary** — a node that replays the primary's log and serves reads.
- **Asynchronous replication** — the primary commits without waiting for the replica.
- **Synchronous replication** — the primary waits until at least one replica has received (or applied) the log before acknowledging.
- **Replication lag** — how far behind the replica is, measured in time or in log position (bytes / LSN / GTID).
- **Read-your-writes (read-after-write) consistency** — a session always sees its own earlier writes.
- **Monotonic reads** — a session never sees data go *backwards* in time (reads from replica A at time T, then replica B which is further behind).
- **LSN / GTID / binlog position** — a monotonically increasing marker of how much log has been written or applied; the tool for "wait until the replica has reached my write".
- **Replay (apply) conflict** — the replica cannot apply a change because a long read query on the replica still needs the old row versions.

### 3. How it works, step by step
1. A client commits a transaction on the primary. The commit record is flushed to the primary's log.
2. A log sender process streams the new log records to each replica over the network (**send** stage).
3. The replica writes them to its own log on disk (**write / flush** stage).
4. A replay/apply process re-executes or re-applies the changes to data pages (**replay / redo** stage). On many engines this stage is *single-threaded* per database unless you enable parallel apply.
5. Only now can a query on the replica see the change.

Lag can accumulate at any of the three stages, and the stage tells you the cause:

| Stage that is slow | Typical cause | How you see it |
|---|---|---|
| Send | Network bandwidth/latency, huge bursts of log (bulk load) | Primary shows a large "sent but not received" gap |
| Write/flush | Replica disk slower than primary's | Received position ahead of flushed position |
| Replay | Single-threaded apply, one huge transaction, replay blocked by long replica queries, replica CPU/IO saturated | Flushed position far ahead of replayed position |

Replay is the usual culprit. A primary can generate changes with 32 concurrent sessions; a replica that replays them serially cannot keep up if the workload is write-heavy. A single transaction that updates 50 million rows is also applied as one unit, and the replica shows nothing until it finishes, then lags by the whole duration.

### 4. Worked example
Measure lag in PostgreSQL (run on the primary):

```sql
SELECT application_name,
       state,
       pg_wal_lsn_diff(pg_current_wal_lsn(), sent_lsn)   AS send_gap_bytes,
       pg_wal_lsn_diff(sent_lsn, write_lsn)              AS write_gap_bytes,
       pg_wal_lsn_diff(write_lsn, flush_lsn)             AS flush_gap_bytes,
       pg_wal_lsn_diff(flush_lsn, replay_lsn)            AS replay_gap_bytes,
       write_lag, flush_lag, replay_lag
FROM   pg_stat_replication;
```

On the replica:

```sql
SELECT now() - pg_last_xact_replay_timestamp() AS apparent_lag;
```

(The second query overstates lag when the primary is idle, because no new transaction has been replayed.)

Equivalents:

- MySQL 8: `SHOW REPLICA STATUS` → `Seconds_Behind_Source`, and the performance_schema `replication_applier_status_by_worker` table for per-worker lag.
- SQL Server Always On: `sys.dm_hadr_database_replica_states` → `log_send_queue_size`, `redo_queue_size`, `redo_rate`; estimated catch-up time ≈ `redo_queue_size / redo_rate`.

Now the *application* problem. A user saves a profile then is redirected to a page that reads from a replica 40 seconds behind. Three standard fixes:

```sql
-- Fix A (primary pinning): application sets a "read from primary until" timestamp
-- in the session/cookie after any write (e.g. now() + 5 seconds). No SQL needed.

-- Fix B (wait for position, PostgreSQL): after the write, capture the primary's LSN...
SELECT pg_current_wal_lsn();            -- on primary, returns '0/3A5F2C8'

-- ...then on the replica, before reading, check we have replayed past it:
SELECT pg_wal_lsn_diff(pg_last_wal_replay_lsn(), '0/3A5F2C8') >= 0 AS caught_up;
-- If false: poll briefly, or fall back to the primary.

-- Fix C (synchronous apply for chosen transactions):
SET LOCAL synchronous_commit = 'remote_apply';
UPDATE profiles SET display_name = 'Mila' WHERE user_id = 42;
-- COMMIT returns only after the synchronous standby has made the change visible.
```

MySQL's equivalent of Fix B is `SELECT WAIT_FOR_EXECUTED_GTID_SET('uuid:1-12345', 1);` on the replica, which blocks up to one second until that GTID set is applied and returns 0 on success.

### 5. Under the hood
*Why replay conflicts happen (PostgreSQL physical replication).* The primary may vacuum away old row versions that no transaction there needs. The replica, which replays the same vacuum record, would remove rows that a long-running *replica* query is still reading. The engine must choose: cancel the query (`max_standby_streaming_delay` expires, error "canceling statement due to conflict with recovery") or pause replay (lag grows). `hot_standby_feedback = on` tells the primary about the replica's oldest snapshot so vacuum holds back, trading replica staleness for bloat on the primary.

*Why MySQL lags.* Statement/row events are applied by a SQL thread. Since 5.7/8.0, `replica_parallel_workers` with `replica_parallel_type=LOGICAL_CLOCK` applies independent transactions concurrently, using commit timestamps from the source to decide which are independent. Group commit on the source is what makes that parallelism available.

*SQL Server.* The secondary's redo thread applies log blocks; with readable secondaries, queries run under snapshot isolation using row versions in tempdb, so a long query on a secondary can also hold up the ghost-cleanup / version-store and cause redo blocking.

*Logical replication and CDC.* A warehouse fed by change-data-capture adds more stages (extract, queue, load, merge), so "lag" becomes end-to-end freshness, usually measured with a heartbeat row the source updates every few seconds.

### 6. Performance and design implications
- Decide a **staleness budget per read**, not per system: profile page after edit (0 s), product listing (5 s), analytics (minutes).
- Replicas do not give you more *write* capacity. Heavy writes make replicas lag more, since replay is the bottleneck.
- Keep long reports off the replica that serves latency-sensitive reads, or give reports their own replica with relaxed conflict settings.
- Break huge transactions into batches (10k–100k rows) so replicas replay in steady increments and you never see "frozen then jump" lag.
- Alert on lag in *bytes* and *seconds*, and on replica replay stopped (lag growing while primary is busy).
- Typical healthy numbers: sub-100 ms replay lag within a region; seconds to minutes cross-region or during bulk loads.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `pg_stat_replication` | Per-replica send/write/flush/replay positions and lag | Diagnose which stage is slow |
| `pg_last_xact_replay_timestamp()` | Commit time of last replayed transaction | Quick replica-side lag estimate |
| `synchronous_commit = remote_apply` | Commit waits until replica applied | Read-your-writes for selected transactions |
| `hot_standby_feedback` | Replica tells primary about its oldest snapshot | Stop query cancellations on a reporting replica |
| `max_standby_streaming_delay` | How long replay may wait for conflicting queries | Balance cancelled queries vs lag |
| `SHOW REPLICA STATUS` (MySQL) | Lag, thread state, errors | First look on MySQL |
| `WAIT_FOR_EXECUTED_GTID_SET` | Block until a GTID is applied | Read-after-write on MySQL replicas |
| `replica_parallel_workers` | Parallel apply threads | Replay-bound lag on MySQL |
| `sys.dm_hadr_database_replica_states` | Send/redo queues on Always On | SQL Server lag diagnosis |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| **Route critical reads to primary** — simple and always correct | Too many "critical" reads defeats the point of replicas |
| **Session pinning for N seconds after a write** — cheap, no engine support needed | N is a guess; a lag spike beyond N still shows stale data |
| **Wait for LSN/GTID token** — exact, replica load stays on replicas | Needs plumbing of the token between write and read paths; can add latency |
| **Synchronous / remote_apply commits** — strongest guarantee | Every such commit pays a network round trip; a dead standby can stall writes |
| **Accept staleness** (dashboards, feeds) — cheapest | The user can see a state they already changed |

### 9. Common mistakes
- Assuming "replica = same data" → users see their edits disappear → decide per-query consistency and implement read-your-writes.
- Reading lag only as `Seconds_Behind_Source` → misleading on an idle primary or with a stalled thread → also watch log positions and a heartbeat.
- Running a 30-minute report on the replica used for the app → replay conflicts/lag spikes → dedicated reporting replica or `hot_standby_feedback` with bloat monitoring.
- Letting a nightly job update 100 M rows in one transaction → replica "freezes" then jumps → batch in chunks.
- Load-balancing a session's reads across replicas of different lag → data goes backwards (non-monotonic reads) → sticky replica per session.
- Failing over to a lagging replica without checking → lost recent commits → check replay position first or use synchronous standby for zero data loss.

### 10. 🎯 Interview kit
**30-second answer:** Async replicas apply the primary's log after the fact, so reads from them can be stale. I measure lag by log position and time, find the slow stage (send, flush, replay), then fix read-after-write by sending just-written-by-this-user reads to the primary for a short window or by waiting for the replica to reach the write's LSN/GTID.

**Structure a longer answer:** clarify (which DB, sync or async, how stale is "stale") → mechanism (log shipping stages, single-threaded replay) → diagnose (views and numbers) → patterns (primary pinning, token wait, remote_apply, sticky replicas) → trade-offs and a concrete scenario.

**Likely follow-ups:**
- *Why does one huge transaction cause lag?* → applied as a unit; replica shows nothing until commit replayed.
- *Why can a replica query be cancelled?* → replay conflict with vacuum/cleanup; settings trade cancellation for lag.
- *Does synchronous replication remove lag?* → "received" is not "applied"; only `remote_apply`-style modes guarantee visibility.
- *How do you do read-after-write without abandoning replicas?* → pin, token-wait, or synchronous for selected writes.

**Signals interviewers look for:** you separate send/flush/replay, you give a per-read consistency policy, you know the operational causes (long reports, big transactions, single-threaded apply), and you can name the exact catalog views.

### 11. Check yourself
- [ ] Draw the path of a commit from primary to replica and mark the three places lag can build up.
- [ ] Write the query that shows replay lag in bytes on your engine.
- [ ] Explain why `hot_standby_feedback` can cause bloat on the primary.
- [ ] Design a read-after-write scheme using an LSN/GTID token.
- [ ] Explain monotonic reads and how sticky replica routing provides them.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [A query is fast in staging but times out in prod with identical data volume. What environment differences (stats, config, parameterization, cache warmth) would you check first?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39936_A_query_is_fast_in_staging_but_times_out_in_prod_with_identi.md>)
- [Intermediate] [How would you compute a per-source freshness metric (minutes since latest loaded row) across dozens of tables in one warehouse query so a single dashboard shows every pipeline's staleness?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40167_How_would_you_compute_a_per-source_freshness_metric_minutes_.md>)
- [Deep] [How would you design a materialized view for a heavy dashboard query, keep it fresh with REFRESH CONCURRENTLY, and surface the staleness/consistency trade-off to product?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39954_How_would_you_design_a_materialized_view_for_a_heavy_dashboa.md>)
- [Deep] [A downstream analyst's query re-scans your raw events table 40 times a day. How would you decide between a materialized view, a scheduled aggregate table, and a warehouse result cache to cut cost?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40123_A_downstream_analyst_s_query_re-scans_your_raw_events_table_.md>)
- [Deep] [Your read replica is 40 seconds behind primary and users are complaining they don't see their just-saved data. How do you diagnose the lag source and what patterns fix read-after-write without abandoning replicas?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40280_Your_read_replica_is_40_seconds_behind_primary_and_users_are.md>)
- [Deep] [A report needs the current balance for millions of accounts derived from a transactions table. Compare recomputing from events, a maintained balance column with triggers, and periodic snapshots. Which and why?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40381_A_report_needs_the_current_balance_for_millions_of_accounts_.md>)
- [Deep] [Your semantic cache is returning stale answers after documents changed. How would you invalidate cache rows in SQL when the underlying source chunk's version bumps?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40852_Your_semantic_cache_is_returning_stale_answers_after_documen.md>)
- [Expert] [How would you back up and restore a multi-terabyte database with an RPO of 5 minutes and RTO of 1 hour? Explain WAL archiving, PITR, and how you'd actually test the restore.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40360_How_would_you_back_up_and_restore_a_multi-terabyte_database_.md>)

## Related concepts

- [Index Strategy and Governance at Scale](<../../Query_Performance/Indexing/Index_Tuning_in_Practice/Index_Strategy_and_Governance_at_Scale.md>) *(Indexing)*
- [Read Your Writes in MySQL](<../../../TOOLS/MySQL/MySQL_Platform/Read_Your_Writes_in_MySQL.md>) *(MySQL Platform)*
- [Replica Lag and Read Your Writes](<../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Replica_Lag_and_Read_Your_Writes.md>) *(PostgreSQL Platform)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Troubleshooting Poor Performance Caused by Stale Statistics
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Advanced Best Practices Recap
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Asynchronous Statistics Updates
- *Data Engineering with Medallion Architecture* — Miki Eto — Scaling Data Platforms with Optimization › Concrete example
- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioned Logs › Replaying old messages
- *Designing Data-Intensive Applications* — Martin Kleppmann — Databases and Streams › Initial snapshot
- *Designing Data-Intensive Applications* — Martin Kleppmann — Observing Derived State › Pushing state changes to clients
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Monitoring and Anomaly Detection for Your Data Pipelines › Monitoring for Freshness
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — A Data Architecture Sampler › Data integrity
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Aggregations with Event-Time Windows
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Building Reliable Data Lakes with Apache Spark › Querying Previous Snapshots of a Table with Time Travel
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — T › W

