---
id: "01_Databases_and_SQL/Database_Operations/Replication/Replication_Topologies_and_Lag"
level: 5
type: "deep-dive"
topic: "01_Databases_and_SQL/Database_Operations/Replication"
tools: ["MySQL"]
---

# Replication Topologies and Lag

**L5 Expert** · [↑ Replication](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Replication copies a primary's change log to other nodes; the topology decides who may write and how far copies can fall behind, and the sync mode decides how much committed data you can lose when the primary dies.

### 1. The big picture
A single database server is a single point of failure and a single ceiling for read throughput. Replication keeps several copies of the same data on different machines so that (a) one copy can take over when another fails, (b) reads can be spread across copies, and (c) heavy jobs such as backups and analytical scans can run somewhere other than the production primary.

Every design choice in replication trades three things against each other: **durability** (do acknowledged writes survive a crash?), **latency** (how long does a commit wait?) and **freshness** (how stale can a replica be?). Topology (single-leader, chained, multi-leader) and sync mode (asynchronous, semi-synchronous, synchronous) are the two dials. *Replication lag* is the observable symptom of choosing the cheap end of those dials. This lesson is the foundation for the failback and schema-change lessons that follow in this topic.

### 2. Core concepts and vocabulary
- **Primary / leader / master** — the node that accepts writes and produces the change log.
- **Replica / follower / slave** — a node that applies the primary's change log; usually read-only.
- **Change log** — the ordered record of changes: MySQL's *binary log* (binlog), PostgreSQL's *WAL*, SQL Server's *transaction log*. Replication ships this log, not the tables.
- **Statement-based vs row-based** — statement-based logs the SQL text (small, but non-deterministic functions can diverge); row-based logs the before/after row images (verbose, but exact); *mixed* switches per statement.
- **Relay log** — on a MySQL replica, the local copy of the primary's binlog that the apply thread reads from.
- **I/O thread and SQL (applier) thread** — the receiver that fetches events, and the applier that replays them. They fail and lag independently.
- **Asynchronous / semi-synchronous / synchronous** — whether the primary waits for zero, at least one, or all replicas to acknowledge before telling the client "committed".
- **Replication lag** — the time (or log distance) between a commit on the primary and its visibility on a replica.
- **GTID** — a global transaction identifier that lets a replica say "I have applied everything up to X" without file/offset bookkeeping.
- **Quorum** — a majority of copies; if writes and reads each touch a majority, any read overlaps at least one node that saw the latest completed write.
- **Witness replica** — a node that records only the fact that a write happened (not the data) so it can count towards a quorum cheaply.

### 3. How it works, step by step
**Asynchronous replication, end to end**

1. Client sends `COMMIT`. The primary writes the transaction to its own log and flushes it to disk.
2. The primary immediately returns success to the client. It has **not** waited for any replica.
3. A log-sender on the primary streams new events to each connected replica (the replica's I/O thread requests them from a position or GTID).
4. The replica appends the events to its relay log (receiving is done at this point).
5. The SQL/applier thread replays the events, in order (or in parallel by dependency/schema/group-commit in modern engines).
6. The change becomes visible to reads on the replica.

Lag can build up at three places: the network (step 3), the receiver (step 4) and, most commonly, the **applier (step 5)**, because the primary executes transactions concurrently over many cores while a single-threaded applier replays them serially.

| Mode | Primary waits for | Data lost on primary crash | Commit latency | Availability risk |
|---|---|---|---|---|
| Asynchronous | nothing | recent commits not yet shipped | lowest | none from replicas |
| Semi-synchronous | ≥1 replica has *received* (not applied) the log | normally none acknowledged (falls back to async on timeout) | + one round trip | falls back or stalls |
| Synchronous | all (or a quorum of) replicas applied/durable | none | highest, grows with replica count | one slow/dead replica blocks writes |

**Topologies**

- *Single-leader (one primary, N replicas)* — the default. Simple conflict-free ordering; the primary's log is the single ordering authority.
- *Chained / cascading* — replica B replicates from replica A, which replicates from the primary. Cuts fan-out load on the primary; each hop adds lag and a new failure point.
- *Multi-leader (active-active)* — two or more nodes accept writes, typically one per region. Write latency is local, but concurrent writes to the same row **conflict** and must be resolved (last-write-wins by timestamp, version vectors, application merge). Auto-increment keys need `increment`/`offset` settings to avoid collisions.
- *Leaderless / quorum* — any node accepts writes; reads and writes each contact a majority. Witness replicas lower storage cost: with three copy replicas and two witnesses, a failure of copy nodes can temporarily "upgrade" a witness to hold the data, reverting once the copy node returns.
- *Delayed replica* — a deliberate lag (e.g. one hour). It is not a backup (it will faithfully apply a `DROP TABLE` once the delay expires) but it gives a window to stop replication before a logical mistake is applied.

### 4. Worked example
Measuring lag on a MySQL 8 replica:

```sql
-- On the replica
SHOW REPLICA STATUS\G
-- Look at:
--   Replica_IO_Running: Yes        (receiver connected?)
--   Replica_SQL_Running: Yes       (applier alive?)
--   Seconds_Behind_Source: 37      (rough lag; NULL means a thread is stopped)
--   Retrieved_Gtid_Set vs Executed_Gtid_Set   (received but not yet applied)
--   Last_SQL_Error: ...            (why the applier stopped)
```

`Seconds_Behind_Source` is computed from event timestamps and **lies** in common cases: it is `0` while the I/O thread is disconnected (nothing is queued, so nothing looks late), and it jumps around with large transactions. A reliable method is a **heartbeat table**: a job on the primary updates `heartbeat(ts)` every second; on the replica you read `NOW() - ts`. That measures end-to-end staleness including network and apply.

Diagnosis pattern: if `Retrieved_Gtid_Set` is far ahead of `Executed_Gtid_Set`, the receiver is fine and the **applier** is the bottleneck (long transactions, no primary keys forcing full scans per row event, single-threaded apply, or a weaker replica box). If both are stuck equally, suspect the network or the primary.

`MASTER_HEARTBEAT_PERIOD` (`SOURCE_HEARTBEAT_PERIOD`) makes the primary send a tiny heartbeat when it has no events, so a replica can distinguish "primary is idle" from "connection is dead" and doesn't have to wait for `net_read_timeout` to notice.

### 5. Under the hood
- **Ordering.** Replicas must apply conflicting changes in the primary's commit order. Parallel appliers get dependency information from the primary (group-commit timestamps or write-set hashes) so non-conflicting transactions can run concurrently while still committing in a consistent order.
- **Row events without a primary key.** A row-based `UPDATE` on a table with no PK forces the applier to locate each row by scanning; one statement touching 100k rows becomes 100k scans. This is a classic cause of runaway lag and is why "no PK-less tables" is a fleet policy.
- **Semi-sync mechanics.** The primary blocks the committing thread after the log is written and releases it when a replica's I/O thread ACKs receipt (not apply). If no ACK arrives within a timeout, the primary silently degrades to asynchronous — so "semi-sync" is a best-effort guarantee unless you configure it to wait indefinitely.
- **Quorum arithmetic.** With replication factor *N*, write quorum *W* and read quorum *R*, setting *W + R > N* guarantees overlap, so a quorum read sees the latest completed quorum write.
- **CDC vs native replication.** Native replication ties replica and primary to the same engine and version family and gives an exact copy. Change-data-capture reads the same log but emits events into a stream, so targets can be a different database, object storage and an analytics processor at once — loosely coupled, asynchronous, and slightly delayed.

### 6. Performance and design implications
- Replication adds write amplification on the primary's log and network; sync modes add a round trip to every commit (hundreds of microseconds in a rack, tens of milliseconds across regions).
- Read scaling is only valid if the application tolerates stale reads. Anything that reads its own writes (profile update then reload) needs **sticky routing** to the primary for a short window, or a GTID-wait ("wait until replica has applied transaction X").
- Provision replicas at least as strong as the primary; a smaller replica keeps up in steady state and collapses under a spike because the applier has no headroom.
- Alert on lag in seconds **and** on thread state, and track relay-log disk usage (a stopped applier lets the relay log grow without bound).
- Run backups and long scans on a replica, but remember a lagged replica gives a backup that is older than you think; record the GTID/binlog position with the backup.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SHOW REPLICA STATUS` | Thread state, lag, errors, GTID sets | First stop for any lag/stop incident |
| `read_only` / `super_read_only` | Block writes on a replica; `super_read_only` also blocks users with SUPER | Every replica, to prevent accidental writes |
| `binlog_format=ROW` | Exact row images in the log | Default choice for correctness |
| `replica_parallel_workers` | Number of parallel applier threads | Lag caused by single-threaded apply |
| `rpl_semi_sync_*` | Enable and tune semi-sync | Need near-zero loss without full sync cost |
| `CHANGE REPLICATION SOURCE TO … SOURCE_DELAY=n` | Delayed replica | Safety window vs. logical corruption |
| `auto_increment_increment/offset` | Interleave generated keys | Multi-leader topologies |
| pt-heartbeat / heartbeat table | Real end-to-end lag | When `Seconds_Behind` is not trustworthy |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Async single-leader: read scaling, tolerable small data loss | Money-like data where an acknowledged commit must never vanish |
| Semi-sync: want zero acknowledged-loss in the normal case | Cross-region links where each commit would pay WAN latency |
| Full sync/quorum: strong consistency required | Many replicas; one slow node drags everything, availability drops |
| Multi-leader: regional write latency, regional autonomy | You cannot define conflict rules; invariants like uniqueness or balances |
| Delayed replica: cheap undo window | Treating it as a backup |

### 9. Common mistakes
- Reading `Seconds_Behind_Source = 0` as healthy → replica was disconnected → alert on thread state and use a heartbeat table.
- Routing reads blindly to replicas → users see their own updates disappear → sticky/GTID-aware routing.
- PK-less tables with row-based replication → applier scans per row → add primary keys.
- Letting replicas be writable → silent divergence and later duplicate-key failures → `super_read_only`.
- Assuming semi-sync is a hard guarantee → silent fallback to async → monitor the semi-sync status counter.
- Promoting a replica without checking it applied its whole relay log → lost transactions → verify executed GTID set equals received set first.

### 10. 🎯 Interview kit
**30-second answer:** Replication ships the primary's ordered change log to replicas. Async is fastest but can lose the tail on failure; semi-sync waits for one replica to receive; sync waits for all and trades availability and latency for zero loss. Lag is the gap between commit and apply, usually caused by the single-threaded or scan-heavy applier.

**Structure a longer answer:** clarify (RPO, read-staleness tolerance, regions) → core mechanism (log, I/O vs SQL thread, mode) → trade-offs (durability/latency/availability) → close with a real scenario (e.g. payments: semi-sync + GTID-verified promotion; reporting: async replicas with sticky reads).

**Likely follow-ups:**
- *How do you measure lag reliably?* → heartbeat table; `Seconds_Behind` is misleading when disconnected.
- *Why does lag grow while the primary is calm?* → large transactions, PK-less row events, applier parallelism, weak replica hardware.
- *How would you handle active-active across regions?* → avoid if possible; otherwise partition ownership of rows per region, define deterministic conflict rules, protect unique/balance invariants outside the replicated path.
- *Replication vs backup?* → replication copies mistakes instantly; only backups/PITR give history.

**Signals interviewers look for:** distinguishing received vs applied, naming failure modes (silent semi-sync fallback, split-brain), proposing measurement not guesses.

### 11. Check yourself
- [ ] Draw the path of one write from client commit to visibility on an async replica, marking where lag can accumulate.
- [ ] Explain why `Seconds_Behind_Source` can read 0 during an outage.
- [ ] State the durability/latency/availability trade-off of async, semi-sync and sync in one table.
- [ ] Explain why W + R > N gives read-your-latest-write, and what a witness replica changes.
- [ ] Diagnose: I/O thread running, SQL thread running, lag growing for hours. List three causes and the check for each.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../../../../TOOLS/MySQL/_index.md>)

## Practice questions

- [Intermediate] [What is replication lag and how would you measure it on a MySQL replica in production?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Replication/Q-21170_What_is_replication_lag_and_how_would_you_measure_it_on_a_My.md>)
- [Deep] [Compare asynchronous, semi-synchronous, and synchronous replication in terms of durability and write latency trade-offs.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Replication/Q-21179_Compare_asynchronous_semi-synchronous_and_synchronous_replic.md>)
- [Deep] [What is the role of MASTER_HEARTBEAT_PERIOD and how does it help distinguish a stalled replica from an idle master?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Replication/Q-21224_What_is_the_role_of_MASTER_HEARTBEAT_PERIOD_and_how_does_it_.md>)
- [Deep] [What logging and observability would you add to trace a single write from master commit through to visible-on-replica for debugging?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Replication/Q-21268_What_logging_and_observability_would_you_add_to_trace_a_sing.md>)
- [Deep] [How do you use a delayed replica as an organizational safety net against logical corruption, and what's your runbook for extracting from it under pressure?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Replication/Q-21333_How_do_you_use_a_delayed_replica_as_an_organizational_safety.md>)
- [Expert] [You need zero-downtime major version upgrade of MySQL. How do you use replication to upgrade replicas first then fail over?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Replication/Q-21238_You_need_zero-downtime_major_version_upgrade_of_MySQL_How_do.md>)
- [Expert] [How do you prevent the classic dual-write inconsistency between the primary and downstream stores by treating the replication log as the single ordering authority?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Replication/Q-21292_How_do_you_prevent_the_classic_dual-write_inconsistency_betw.md>)
- [Expert] [How would you design conflict resolution if the org insists on active-active writes across regions despite your reservations, and what invariants do you protect?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Replication/Q-21329_How_would_you_design_conflict_resolution_if_the_org_insists_.md>)

## Related concepts

- [Zero Downtime Sharding Migrations](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Zero_Downtime_Sharding_Migrations.md>) *(Distributed Systems Core)*
- [Replica Lag and Read Your Writes](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Replica_Lag_and_Read_Your_Writes.md>) *(PostgreSQL Platform)*
- [Read Your Writes in MySQL](<../../../../TOOLS/MySQL/MySQL_Platform/Read_Your_Writes_in_MySQL.md>) *(MySQL Platform)*
- [HA Failover Design](<../../High_Availability/HA_Failover_Design.md>) *(High Availability)*
- [Sync vs Async Replication](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Sync_vs_Async_Replication.md>) *(Distributed Systems Core)*
- [Read Your Writes with Replicas](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Read_Your_Writes_with_Replicas.md>) *(Cloud Fundamentals)*

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 5. Replication › Leaders and Followers
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 5. Replication › Synchronous Versus Asynchronous Replication
- *Designing Data-Intensive Applications* — Martin Kleppmann — Setting Up New Followers
- *Designing Data-Intensive Applications* — Martin Kleppmann — Setting Up New Followers › Implementation of Replication Logs
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ingestion Design Patterns › Replication
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ingestion Design Patterns › Pattern: Passthrough Replicator
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ingestion Design Patterns › Solution
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ingestion Design Patterns › Security and isolation
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › CDC and database replication
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Operational Visibility › Replication State
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Data Storage, Indexing, and Replication › Replication models
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Data Storage, Indexing, and Replication › Replication log formats
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Data Storage, Indexing, and Replication › Row-based replication
- *Think Distributed Systems* — Dominik Tornow — Replication › 8.3 Replication
- *Think Distributed Systems* — Dominik Tornow — Replication › 8.4.4 State-based vs. log-based replication
- *Think Distributed Systems* — Dominik Tornow — Distributed consensus › 10.6.4 Log Replication protocol
- *Ultimate AWS Data Engineering* — Rathish Mohan — Architecting for Scalability Data Replication Techniques › Introduction to Data Replication
- *Ultimate AWS Data Engineering* — Rathish Mohan — Architecting for Scalability Data Replication Techniques › Example: Real-World Application of Data Replication in AWS
- *Ultimate AWS Data Engineering* — Rathish Mohan — Architecting for Scalability Data Replication Techniques › Monitoring and Managing Data Replication
- *Ultimate AWS Data Engineering* — Rathish Mohan — Architecting for Scalability Data Replication Techniques › Synchronous Replication: Strong Consistency at a Cost

