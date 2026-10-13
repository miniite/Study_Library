---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/02_Replication_and_Partitioning/Leader_Follower_Replication/Replication_Logs"
level: 5
type: "curriculum"
curriculum: "data_engineering"
---

# Replication Log Implementations

**L5 Expert** · [↑ Leader-Follower Replication](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A leader ships its changes to followers as a replication log, and the log's format (statements, WAL bytes, logical row events, or triggers) determines what the followers can be, how portable it is, and which bugs you will meet.

### 1. The big picture
Leader-based replication needs a stream of changes that every follower can apply in the same order. There are four implementations of that stream, differing in how abstract the entries are. The more physical the log (bytes of disk pages), the faster and more exact; the more logical (row changes), the more it decouples replication from storage engine internals and enables version upgrades, heterogeneous targets and change data capture (CDC).

It helps to know the write-ahead log (WAL) first: the append-only, sequential, immutable file in which the database records every change *before* applying it to data pages, giving crash recovery. Each record has a monotonically increasing **log sequence number (LSN)**; a transaction is committed only after the log is forced to disk up to its commit record's LSN. Replication builds on this same log.

### 2. Core concepts and vocabulary
- **WAL (write-ahead log)** — append-only log of changes written before data pages; basis of durability.
- **LSN** — log sequence number; position in the log.
- **Force** — flushing log buffers to disk in LSN order.
- **Checkpoint** — marker that all changes up to an LSN are persisted in the main structure so older log can be trimmed. *Fuzzy checkpoints* flush pages asynchronously between `begin_checkpoint` and `end_checkpoint` records.
- **Statement-based replication** — ship the SQL statements.
- **WAL shipping (physical replication)** — ship the storage engine's byte-level log.
- **Logical (row-based) log** — ship row-level changes: new row values for inserts, old+new or key for updates/deletes.
- **Trigger-based replication** — application-level mechanism using triggers writing to a change table.
- **CDC (change data capture)** — consuming the logical log as an event stream.

### 3. How it works, step by step
**Statement-based**: leader logs each `INSERT/UPDATE/DELETE` and followers re-execute it. Breaks down on:
1. Nondeterministic functions (`NOW()`, `RAND()`, `UUID()`) giving different values.
2. Autoincrement or order-dependent statements (`UPDATE … WHERE`), which require identical execution order.
3. Side effects (triggers, stored procedures, UDFs) not deterministic.
Workaround: leader replaces nondeterministic calls with literal values; MySQL pre-5.1 used this and later defaulted to row-based when needed.

**WAL shipping**: leader sends the exact append-only byte sequence (PostgreSQL, Oracle). Followers apply the same page-level operations, producing an identical on-disk copy. Disadvantage: the log describes *which bytes of which disk blocks* changed, tightly coupled to engine version, so leader and follower usually must run the same version; zero-downtime upgrade by replica-first rolling is hard.

**Logical (row-based) log**: a separate log format decoupled from storage:
1. Insert → all column values of the new row.
2. Delete → enough to identify the row (primary key or all old values).
3. Update → identity plus new values.
4. A transaction produces several records followed by a commit record. MySQL binlog (row format) works like this.
Advantages: backward compatible across versions, can feed external systems (warehouse, caches, search indexes), enables CDC.

**Trigger-based**: triggers or application code capture changes into a table that a process replicates; flexible (replicate a subset, transform, conflict resolution) but higher overhead and more bug-prone (Databus for Oracle, Londiste for PostgreSQL).

### 4. Worked example
```sql
-- PostgreSQL logical decoding: see row-level changes as a log
SELECT pg_create_logical_replication_slot('demo', 'test_decoding');
INSERT INTO orders(id, status) VALUES (1, 'new');
UPDATE orders SET status = 'paid' WHERE id = 1;
SELECT lsn, xid, data
FROM   pg_logical_slot_get_changes('demo', NULL, NULL);
-- BEGIN 7421
-- table public.orders: INSERT: id[integer]:1 status[text]:'new'
-- table public.orders: UPDATE: id[integer]:1 status[text]:'paid'
-- COMMIT 7421
```
Reading it: each change is a logical row event, ordered by LSN and grouped by transaction. A consumer such as Debezium reads exactly this stream and publishes it to Kafka.

### 5. Under the hood
- **Slots and retention**: a logical/physical replication slot makes the leader retain WAL until the consumer confirms it. A stalled follower or CDC consumer therefore fills the leader's disk; monitor slot lag.
- **Trim and checkpoint coupling**: the WAL can only be truncated up to the last completed checkpoint and the slowest replica's confirmed LSN.
- **Compensation log records** (CLRs) record undo actions so a crash during rollback is still recoverable; replication must preserve them in physical streams.
- **Ordering**: logs establish a total order per leader; followers apply serially (or in parallel by transaction dependencies in MySQL multi-threaded replication), which determines catch-up speed.
- **Heterogeneity**: row-based logs let you replicate between different storage engines or database vendors, but with DDL, large-object and ordering caveats.

### 6. Performance and design implications
- Physical WAL shipping is cheapest on the leader and exact, but replicas must be same version and cannot filter tables.
- Logical logs are larger and cost CPU to decode; however they enable selective replication and CDC pipelines.
- Row-based logging of a bulk `UPDATE` produces a huge log; statement-based would be a single line.
- Slow single-threaded apply is a classic source of replica lag.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `wal_level = logical` (PostgreSQL) | Enables logical decoding | CDC, logical replication |
| `binlog_format = ROW/STATEMENT/MIXED` (MySQL) | Chooses log format | Correct replication; CDC needs ROW |
| `pg_replication_slots` view | Shows slot lag and retained WAL | Prevent disk fill |
| `CREATE PUBLICATION / SUBSCRIPTION` | Logical replication in Postgres | Partial or cross-version replication |
| Debezium connectors | Convert logical logs to Kafka events | Streaming CDC |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| WAL shipping: HA standby of same version, simplest and fast | Need cross-version, partial or heterogeneous targets |
| Logical/row-based: CDC, upgrades, selective replication | Very large batch updates dominate |
| Statement-based: tiny logs, deterministic workloads | Any nondeterminism or triggers |
| Trigger-based: custom filtering/conflict logic | You need performance and reliability at scale |

### 9. Common mistakes
- Statement-based replication with `NOW()`/`RAND()` → replicas diverge → use row-based or mixed.
- Ignoring an inactive replication slot → WAL fills disk, leader crashes → alert and drop.
- Running different major versions on physical replicas → cannot apply log → use logical replication or upgrade followers first only when supported.
- Forgetting that a tiny `UPDATE … WHERE` may touch millions of rows in a row log → lag spike.

### 10. 🎯 Interview kit
**30-second answer:** The replication log is the leader's ordered change stream. Statement-based ships SQL (fragile with nondeterminism), WAL shipping ships physical page changes (fast but version-coupled), logical row-based decouples from storage and powers CDC, and trigger-based is custom application-level replication.

**Structure a longer answer:** WAL and LSN basics → the four formats with failure modes → coupling to storage engine and upgrades → CDC use → operational risks (slots, lag).

**Likely follow-ups:**
- *Why does MySQL default to row-based?* → statement-based is nondeterministic-unsafe.
- *How does Debezium work?* → reads the logical log through a replication slot and emits events to Kafka.
- *What is an LSN used for?* → ordering, position tracking, and snapshot + catch-up.

**Signals interviewers look for:** knowing each method's failure mode, connecting WAL to replication and CDC.

### 11. Check yourself
- [ ] List three ways statement-based replication diverges.
- [ ] Explain why WAL shipping makes rolling upgrades hard.
- [ ] Describe what a logical record for an update contains.
- [ ] Explain the danger of an abandoned replication slot.
<!-- explainer:end -->

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Setting Up New Followers › Logical (row-based) log replication
- *Designing Data-Intensive Applications* — Martin Kleppmann — Setting Up New Followers › Trigger-based replication
- *Designing Data-Intensive Applications* — Martin Kleppmann — Setting Up New Followers › Write-ahead log (WAL) shipping
- *Designing Data-Intensive Applications* — Martin Kleppmann — Setting Up New Followers › Implementation of Replication Logs
- *Designing Data-Intensive Applications* — Martin Kleppmann — S
- *Database Internals* — Alex Petrov — Transaction Processing and Recovery › Log Semantics
- *Designing Data-Intensive Applications* — Martin Kleppmann — W

