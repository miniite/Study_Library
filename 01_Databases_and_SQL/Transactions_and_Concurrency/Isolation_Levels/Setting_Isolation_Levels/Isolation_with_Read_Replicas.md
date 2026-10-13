---
id: "01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Isolation_with_Read_Replicas"
level: 5
type: "deep-dive"
topic: "01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels"
tools: []
---

# Isolation with Read Replicas

**L5 Expert** · [↑ Isolation Levels](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Isolation levels describe what concurrent transactions on *one* copy of the data can see; the moment reads move to a replica, a second, separate question appears — how stale and how ordered is that copy? — and isolation alone cannot answer it.

### 1. The big picture
A transaction's isolation level (Read Committed, Repeatable Read, Snapshot, Serializable) is a promise made by **one database instance** about how concurrent transactions interleave. Replication adds another dimension: the same logical data lives on several machines, and changes reach the copies after a delay. The guarantees you get from a replica are therefore a combination of two independent things:

- **Isolation** — inside the replica, does a query see a consistent snapshot (no torn reads across tables, no dirty data)?
- **Replication consistency** — how far behind the primary is that snapshot, and do successive reads by the same user ever go *backwards* in time?

A replica can give you a perfectly isolated, internally consistent snapshot that is nevertheless 8 seconds old. That is "consistent but stale", and it is a valid, useful state — but it is not the same as "current". Most production bugs in read-scaled systems come from teams treating the two ideas as one: they chose Serializable on the primary and assumed every read, from anywhere, inherits that promise. It does not.

This lesson connects the isolation levels you already know to the consistency models used for distributed copies, and shows how to design read routing so that the guarantee users actually experience is the one you intended.

### 2. Core concepts and vocabulary
- **Concurrency control** — the set of techniques (locks, snapshots, versioning) a database uses to make concurrent transactions behave as if they ran one after another.
- **Isolation** — the "I" in ACID: the result of concurrent transactions equals some serial execution (strictly only at Serializable; weaker levels permit named anomalies).
- **Pessimistic vs optimistic control** — pessimistic takes locks up front and makes others wait; optimistic lets everyone proceed and detects a conflict at commit (for example a version column or snapshot write-conflict check).
- **Replica / secondary / standby** — a copy kept current by applying the primary's log (physical log shipping, logical/row replication, or binlog).
- **Replication lag** — time (or log distance) between a commit on the primary and its visibility on a replica. Asynchronous replication has non-zero, variable lag; synchronous replication waits for the replica to harden the log before acknowledging the commit.
- **Stale read** — a read that returns a state that was true earlier but has since been superseded.
- **Strict consistency** — every write is instantly visible everywhere; requires a perfect global clock and is physically unattainable.
- **Linearizability** — the strongest *achievable* single-object model: each write takes effect atomically at one point between its start and end, and once any client has seen it, no client ever sees the older value again.
- **Session guarantees** — weaker, cheap, per-client promises: *read-your-writes*, *monotonic reads* (never go back in time), *monotonic writes*, *writes-follow-reads*.
- **Consistent prefix** — a reader sees an initial segment of the primary's history, never a later change without an earlier one it depended on.
- **External consistency** — the order of transactions as observed by the outside world matches commit order in real time; systems like Spanner achieve it with bounded-uncertainty clocks (TrueTime) or hybrid logical clocks plus commit-wait.

### 3. How it works, step by step
1. **Primary commits.** The transaction's changes are written to the transaction log (WAL/binlog/redo) and the commit is acknowledged to the client. Under any isolation level the primary enforces its rules here.
2. **Log shipping.** The log records stream to each replica. In asynchronous mode the primary does not wait.
3. **Replay on the replica.** A replay/redo worker applies the changes. Until a record is replayed, queries on the replica do not see it.
4. **Reader takes a snapshot.** Readers on the replica use the engine's snapshot mechanism (MVCC read view). The snapshot is consistent *with respect to replay position*: it reflects all transactions replayed up to some point and none after.
5. **Result returned.** The application gets a consistent but possibly old answer.

Where the guarantees break:

| Scenario | What the user observes | Violated guarantee |
|---|---|---|
| Write on primary, immediately read on replica | Own write missing | Read-your-writes |
| Two reads routed to two replicas with different lag | Second read older than first | Monotonic reads |
| Comment replicated before the post it replies to (multi-shard) | Answer without question | Consistent prefix |
| Fraud check reads replica, then writes primary | Decision based on old balance | Isolation of the *whole* transaction |
| Report joins table A from replica 1 and table B from replica 2 | Torn cross-table view | Snapshot consistency |

The last two rows matter most: **a read-modify-write transaction whose read half is served from a replica is not isolated at all**, no matter what level the primary runs at. The write half validates against the primary's state; the read half never did.

### 4. Worked example
A payment service reads an account balance from a replica and then debits on the primary.

```sql
-- On the replica (async, 3 s behind)
SELECT balance FROM accounts WHERE id = 42;        -- returns 500

-- Meanwhile on the primary, another request already debited 450:
UPDATE accounts SET balance = balance - 450 WHERE id = 42;   -- committed, balance now 50

-- App logic: 500 >= 300, so approve; run on the primary:
UPDATE accounts SET balance = balance - 300 WHERE id = 42;   -- balance -250 (!)
```

Even if the primary is Serializable, this ran as two separate transactions on two different machines. Fixes, from cheapest to strongest:

```sql
-- Fix A: do the check and the write in ONE primary transaction
BEGIN;
UPDATE accounts SET balance = balance - 300
WHERE id = 42 AND balance >= 300;      -- atomic check-and-set
-- rowcount = 0 -> reject
COMMIT;

-- Fix B: route anything that precedes a write to the primary (read-write transaction goes whole to primary)

-- Fix C: wait for the replica to catch up to a known position (PostgreSQL)
-- after the primary write, capture: SELECT pg_current_wal_lsn();   -- e.g. '0/5A3F1B0'
-- on the replica, poll until:
SELECT pg_last_wal_replay_lsn() >= '0/5A3F1B0'::pg_lsn;
```

MySQL equivalent: capture the GTID of your write and call `WAIT_FOR_EXECUTED_GTID_SET('<gtid>', 1)` on the replica; SQL Server availability groups expose the log send/redo queue in `sys.dm_hadr_database_replica_states`.

### 5. Under the hood
**How a replica makes a snapshot.** A hot standby in PostgreSQL replays WAL and serves queries with normal MVCC snapshots. Replay may need to remove or modify rows that a long replica query still needs. The replica must then choose: *delay replay* (lag grows, bounded by `max_standby_streaming_delay`) or *cancel the query* ("canceling statement due to conflict with recovery"). `hot_standby_feedback = on` tells the primary which rows the replica still needs so VACUUM does not remove them — buying fewer cancellations at the cost of bloat on the primary.

**SQL Server readable secondaries.** Queries on a readable secondary of an availability group are automatically run under snapshot isolation; row versions are stored in the secondary's tempdb, and the primary must add 14-byte version tags to modified rows even if you never enabled row versioning there. Pending redo can be blocked by long readers for DDL-like operations, and a long reader inflates the version store.

**Linearizability vs snapshot isolation.** Snapshot isolation only promises a snapshot *at some point in the past* — nothing says it is the latest. Linearizable reads, by contrast, must observe every write completed before the read began. Getting that from a replica requires the replica to prove it is caught up (a leader lease, a read-index round trip as in Raft, or a timestamp barrier). Distributed SQL systems that offer "external consistency" (Spanner-like designs) use bounded clock uncertainty and make writers wait out the uncertainty window ("commit wait") so timestamp order equals real-time order. A hybrid logical clock gives causal ordering cheaply but does not by itself give real-time external consistency.

**Strict consistency** needs a global clock and instantaneous propagation; no real system implements it, so "strong consistency" in vendor documentation almost always means linearizability for single objects or strict serializability for transactions.

### 6. Performance and design implications
- Read replicas scale *read throughput*; they do not scale write throughput and add replay overhead. A heavily written replica may lag precisely when you need it.
- Measure lag as both **time** and **bytes/LSN**; time-based lag reads 0 on an idle primary even if the replica is broken.
- Routing options, from simplest: (1) read-only dashboards and search → replica, tolerate staleness; (2) sticky sessions → same replica per user for monotonic reads; (3) "read from primary for N seconds after a write by this user"; (4) token-based: return an LSN/GTID/timestamp with each write and require the replica to reach it.
- A cache (Redis) in front of a Read Committed database can resurrect anomalies: a stale value cached after a concurrent write, or a cache refill racing an invalidation. Use versioned keys, delete-after-commit invalidation, short TTLs, or write-through from a single writer.
- Set a **staleness SLO** (for example p99 lag < 2 s) and alert on it, and decide per feature what happens when it is breached (fall back to primary, show a banner, fail).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `pg_current_wal_lsn()` / `pg_last_wal_replay_lsn()` | Compare write position vs replay position | Read-your-writes by LSN |
| `hot_standby_feedback`, `max_standby_streaming_delay` | Trade replica query cancellation vs lag/bloat | Long reports on a PostgreSQL standby |
| `WAIT_FOR_EXECUTED_GTID_SET()` | Block until replica has applied a GTID | MySQL read-your-writes |
| `synchronous_commit = remote_apply` | Primary waits until replica has *applied* the commit | Rare, small set of critical reads |
| `READ_COMMITTED_SNAPSHOT`, `ALLOW_SNAPSHOT_ISOLATION` | Versioned reads on the primary | Reporting on live SQL Server data |
| `sys.dm_hadr_database_replica_states` | Log send/redo queue per replica | Monitoring AG lag |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Replica reads for analytics, search, dashboards | The read feeds a decision that is then written (check-then-act) |
| Sticky routing / "primary after write" window | Traffic is huge and primary cannot absorb the post-write reads |
| LSN/GTID wait tokens | Replicas are routinely far behind; waits turn into timeouts |
| Synchronous or quorum replication | Cross-region latency would break the write SLA |
| Snapshot isolation on a primary for consistent reports | Long snapshots bloat undo/version store |

### 9. Common mistakes
- Assuming "Serializable on the primary" covers replica reads → fraud/limit checks use stale data → do the check inside the write transaction on the primary.
- Round-robin reads across replicas → data appears to move backwards → pin a session to one replica or use position tokens.
- Measuring lag only in seconds → silent breakage on idle systems → also track log position difference.
- Cancelling long reports on a standby with no one noticing → add feedback or run reports on a dedicated replica with a large replay delay allowance.
- Reconstructing cross-table reports from several replicas → torn view → one replica, one transaction (snapshot).

### 10. 🎯 Interview kit
**30-second answer:** Isolation governs interleaving on one node; replicas add lag, so a replica read is a consistent snapshot of the past. Anything that reads then writes must run against the primary, and user-visible guarantees like read-your-writes need explicit routing or position tokens.

**Structure a longer answer:** clarify (what read must be fresh?) → core mechanism (log replay, snapshot on replica) → trade-offs (staleness vs load vs latency) → real scenario (balance check, post-then-read).

**Likely follow-ups:**
- *Pessimistic vs optimistic control and how do isolation levels map?* → Locks at RR/Serializable are pessimistic; snapshot/version-column are optimistic with commit-time conflict checks.
- *How do you deliver read-your-writes across regional replicas?* → write returns an LSN/timestamp; reads carry it; replica waits or falls back to primary.
- *What does external consistency add over snapshot isolation?* → real-time ordering across nodes, paid for with clock uncertainty and commit-wait latency.
- *Eventual / read-committed / serializable "tiers" menu?* → expose guarantees by name with cost and latency, not engine knobs.

**Signals interviewers look for:** you separate isolation from replication consistency, you name the specific session guarantee being violated, and you propose a measurable staleness SLO.

### 11. Check yourself
- [ ] Explain why a check-then-write whose check ran on a replica is unsafe under any isolation level.
- [ ] Name the four session guarantees and one failure each.
- [ ] Describe the replay-vs-query conflict on a hot standby and the two knobs that resolve it.
- [ ] Sketch an LSN-token read-your-writes design.
- [ ] Say what snapshot isolation does *not* promise about freshness.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [What is the difference between optimistic and pessimistic concurrency control, and how do isolation levels map onto each approach?](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30194_What_is_the_difference_between_optimistic_and_pessimistic_co.md>)
- [Deep] [How do you keep a materialized-view refresh consistent with concurrent base-table writes? Which isolation level do you run the refresh under?](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30246_How_do_you_keep_a_materialized-view_refresh_consistent_with_.md>)
- [Deep] [Your event-sourced system reconstructs state from a log. How do isolation levels apply (or not) to the projection/read-model builders and where can anomalies sneak in there?](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30100_Your_event-sourced_system_reconstructs_state_from_a_log_How_.md>)
- [Deep] [You're asked to define what 'strong consistency' means in your platform's public API docs. How do you translate isolation-level semantics into promises customers can build on?](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30145_You_re_asked_to_define_what_strong_consistency_means_in_your.md>)
- [Deep] [Explain how you'd design a 'consistency SLA tier' menu (eventual / read-committed / serializable) that product teams choose from, with cost and latency clearly attached to each tier.](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30177_Explain_how_you_d_design_a_consistency_SLA_tier_menu_eventua.md>)
- [Expert] [You inherit a polyglot estate: Postgres RC for OLTP Mongo snapshot for docs and DynamoDB for sessions. How do you reason about end-to-end isolation for a workflow that spans all three?](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30087_You_inherit_a_polyglot_estate_Postgres_RC_for_OLTP_Mongo_sna.md>)
- [Expert] [Design a strategy so that a long analytical snapshot query never causes catastrophic MVCC bloat or replication lag on the write primary at petabyte scale.](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30112_Design_a_strategy_so_that_a_long_analytical_snapshot_query_n.md>)
- [Expert] [How do you architect isolation for a two-phase inventory reservation (hold then confirm) so a crash between phases never leaks reserved-but-never-confirmed stock at scale?](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30143_How_do_you_architect_isolation_for_a_two-phase_inventory_res.md>)

## Related concepts

- [Range Locks and Retries](<../../Locking/Range_Locks_and_Retries.md>) *(Locking)*
- [Isolation Levels in Transactions](<../../ACID_Transactions/Isolation_Levels_in_Transactions.md>) *(ACID Transactions)*
- [Distributed Counters and Consistency](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Distributed_Counters_and_Consistency.md>) *(Distributed Systems Core)*

## References

- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Datastore Field Guide › Isolation
- *Designing Data-Intensive Applications* — Martin Kleppmann — Preventing Lost Updates
- *Designing Data-Intensive Applications* — Martin Kleppmann — Clock Synchronization and Accuracy › Synchronized clocks for global snapshots
- *Designing Data-Intensive Applications* — Martin Kleppmann — Unbundling Databases › Making unbundling work
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Guidelines for Choosing Isolation Levels
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Read Committed Snapshot Isolation (RCSI)
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Enabling Snapshot and RCSI
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Differences between SI and RCSI
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Architecting Your Lakehouse › Snapshot isolation for reads

