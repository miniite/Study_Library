---
id: "01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Serializable_Performance"
level: 5
type: "deep-dive"
topic: "01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels"
tools: []
---

# Serializable Performance

**L5 Expert** · [↑ Isolation Levels](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** SERIALIZABLE gives the strongest isolation — results equal some one-at-a-time order — but it pays for it with extra locking or conflict tracking, so you get lower throughput, more blocking or aborts, and an application that must be able to retry.

### 1. The big picture
Serializable is the only ANSI level that guarantees no anomaly of any kind. It sounds like the obviously safe choice, and teams often set it globally "to be safe." The cost shows up in production: throughput falls, lock waits and deadlocks multiply, or (in snapshot-based engines) transactions abort with serialization failures at a rate that grows quickly with concurrency and contention.

The key idea is that *serializable is a result, not a mechanism*. Different engines reach it differently, and each has its own performance profile:

- **Lock-based** (SQL Server default Serializable, MySQL InnoDB): readers take shared locks and hold them, plus range locks to stop phantoms. The cost is blocking and deadlocks.
- **Serializable Snapshot Isolation, SSI** (PostgreSQL ≥ 9.1): readers don't block, but the engine tracks read/write dependencies and aborts one transaction when it detects a dangerous cycle. The cost is tracking memory and *false-positive* aborts.
- **Snapshot dressed as serializable** (Oracle): its SERIALIZABLE is really snapshot isolation plus first-committer-wins; write skew can still happen.
- **Optimistic file-based** (Delta Lake): writers validate against the transaction log at commit and fail on conflict.

You should leave this lesson able to say what Serializable costs *in the engine in front of you*, how to measure it, and when to use something narrower.

### 2. Core concepts and vocabulary
- **Schedule** — an ordered list of the read/write/commit/abort operations of a set of transactions as the database sees them.
- **Serial schedule** — transactions run completely one after another; trivially correct, no concurrency.
- **Serializable schedule** — interleaved, but equivalent in outcome to *some* serial schedule.
- **Conflict** — two operations on the same item from different transactions where at least one is a write (R-W, W-R, W-W).
- **Precedence (serialization) graph** — nodes are transactions, edge T1→T2 if T1's conflicting operation came first; the schedule is conflict-serializable iff the graph has no cycle.
- **Phantom** — a repeated predicate read returns new rows inserted by another transaction.
- **Write skew** — two transactions read overlapping data and write *different* rows, jointly violating an invariant; snapshot isolation permits it.
- **rw-antidependency** — T1 reads a version that T2 later overwrites (T1 must appear before T2); the building block of SSI.
- **Serialization failure** — the engine aborts a transaction rather than violate serializability (SQLSTATE `40001`; Oracle ORA-08177; SQL Server 3960 for snapshot update conflict; deadlock 1205/`40P01`).
- **Invariant confluence (I-confluence)** — a property of operations that lets replicas run without coordination while still preserving an integrity constraint.

### 3. How it works, step by step
**Lock-based Serializable (SQL Server).**
1. Reading a row takes an S lock *and holds it until commit* (like Repeatable Read).
2. A query with a range predicate takes **key-range locks** on the index keys it scans and on the gap after the last key, so nobody can insert into the range (phantom prevention).
3. Writers take X locks and wait on conflicting S/range locks.
4. Locks release only at commit/rollback (strict two-phase locking).
5. If two transactions hold S on the same row and each tries to upgrade to X, both wait forever → the deadlock monitor kills one.

**SSI (PostgreSQL).**
1. Each transaction works on a snapshot as in Repeatable Read.
2. Reads leave SIREAD "locks" (they never block anyone) at tuple/page/relation granularity.
3. When a write touches something another concurrent transaction read (or vice versa), the engine records an rw-antidependency edge.
4. If a transaction ends up with both an incoming and an outgoing rw edge to concurrent transactions — a "dangerous structure" — and the third member committed first, one of them is aborted at commit or at the next statement.
5. Application receives `40001`, rolls back, and retries.

**Delta Lake.** An append-only write just adds files; an `UPDATE`/`DELETE`/`MERGE` reads the table metadata/log version it started from, and at commit checks whether any concurrent commit touched the same files; if so the write raises a specific concurrent-modification exception and must be retried.

### 4. Worked example
The classic write-skew rule: "at least one doctor must stay on call."

```sql
-- Table: doctors(id, on_call bool). Two doctors on call; both want to leave.

-- Session A                              -- Session B
BEGIN ISOLATION LEVEL SERIALIZABLE;       BEGIN ISOLATION LEVEL SERIALIZABLE;
SELECT count(*) FROM doctors              SELECT count(*) FROM doctors
 WHERE on_call;            -- 2           WHERE on_call;            -- 2
UPDATE doctors SET on_call=false          UPDATE doctors SET on_call=false
 WHERE id = 1;                             WHERE id = 2;
COMMIT;      -- ok                         COMMIT;  -- ERROR 40001: could not serialize
                                          -- access due to read/write dependencies
```

Under Snapshot/Repeatable Read both commits succeed and nobody is on call. Under Serializable one fails; the application must **re-run the entire transaction**, which now sees count = 1 and refuses. Retry skeleton:

```sql
-- application pseudo-loop
for attempt in 1..5:
    BEGIN ISOLATION LEVEL SERIALIZABLE
    ... do work ...
    COMMIT -> success: break
    on 40001/40P01/1205: ROLLBACK; sleep(random_backoff(attempt)); continue
```

In SQL Server the same code under `SERIALIZABLE` does not fail one side at commit; it deadlocks (both hold S range locks, both request X) and the victim receives error 1205.

### 5. Under the hood
- **Cost of lock-based serializable.** Number of locks held grows with rows *read*, not just rows written. Range locks are only fine-grained if there is an **index on the predicate columns**; without one, the engine must lock the whole table scan range, so one query blocks every writer. Lock memory pushes toward lock escalation (row → table).
- **Cost of SSI.** SIREAD locks are kept even after the reader commits, until all overlapping transactions finish; long transactions keep this state alive. When the tracking table fills, granularity is promoted (tuple → page → relation), producing *more false positives*. Read-only transactions are cheap if they can use a "safe snapshot" (`DEFERRABLE`).
- **Abort-rate scaling.** For N concurrent transactions each touching k items out of a hot set of D, the probability a given transaction conflicts is roughly proportional to N·k²/D. Doubling concurrency or access-set size does far more than doubling aborts; that is where the curve turns from "occasional retry" into "retry storm."
- **Oracle.** `SERIALIZABLE` aborts with ORA-08177 if a row you modified was changed by someone committing after your snapshot — but it does not detect pure read/write skew. If you need true serializability, you must add explicit locking (`SELECT … FOR UPDATE`) or materialize the conflict.
- **MySQL InnoDB.** At SERIALIZABLE, plain `SELECT` in an explicit transaction is implicitly turned into a locking read (`FOR SHARE`), so reads block writers.
- **Coordination avoidance.** Distributed systems can skip coordination entirely when operations are I-confluent: each replica runs on a local snapshot and states are merged with a merge function that preserves the invariant. Insert-only counters and "no duplicate key with unique ids from a partitioned generator" qualify; "balance must stay ≥ 0" does not.

### 6. Performance and design implications
- Measure before changing: transactions/s and p99 latency at the current level vs Serializable on production-like data, *at your real concurrency*; single-user tests hide the cost.
- Track `pg_stat_database.xact_rollback`, `deadlocks`, application-level 40001 counters, SQL Server `Lock Waits/sec`, `Number of Deadlocks/sec`, and wait types `LCK_M_RS_S`, `LCK_M_RIn_*` (range locks).
- Shorten transactions, touch rows in a consistent order, keep access sets small, and add indexes that match predicates.
- Use Serializable only around the critical section, not the whole request; keep non-critical reads at Read Committed.
- Retried work burns CPU: a cost review showing 30 % extra CPU from retries is the signal to restructure (narrower transactions, explicit constraint, queue per key) rather than accept.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SET TRANSACTION ISOLATION LEVEL SERIALIZABLE` / `BEGIN ISOLATION LEVEL SERIALIZABLE` | Sets level for session/transaction | Narrow critical section |
| `BEGIN ISOLATION LEVEL SERIALIZABLE READ ONLY DEFERRABLE` (PostgreSQL) | Waits for a safe snapshot, then never aborts | Long reports needing consistency |
| `HOLDLOCK` / `UPDLOCK` table hints | Range + update locks without changing session level | Targeted protection in SQL Server |
| `SELECT … FOR UPDATE` | Explicit pessimistic row lock | Replace "Serializable everywhere" |
| `max_pred_locks_per_transaction` (PostgreSQL) | Capacity for SIREAD tracking | Frequent false positives |
| `READ_COMMITTED_SNAPSHOT` ON | Readers use versions, don't take S locks | Cut reader/writer blocking at RC |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Invariant spans several rows and can't be a constraint | A unique/check/FK constraint or one atomic `UPDATE … WHERE` can enforce it |
| Low contention, short transactions, retry layer exists | Hot rows, long transactions, side effects inside the transaction |
| PostgreSQL SSI for read-heavy mixed workloads | You assume SSI is lock-free for writers' overhead or free of false aborts |
| Explicit locks on a few known critical rows | A newcomer proposes removing all locks "because Serializable covers it" without load testing |
| Single-region deployment | Multi-region active-active — local serializable plus global conflict resolution is *not* global serializability |

### 9. Common mistakes
- Setting the whole app to SERIALIZABLE → throughput collapse, deadlocks → use it surgically, default to Read Committed (+ snapshot) and enforce invariants by constraint.
- No retry logic → user-visible 500s on `40001` → wrap the whole transaction in a bounded retry with jittered backoff.
- Retrying a transaction that emailed or charged a card → duplicates → perform side effects *after* commit via an outbox, or make them idempotent with a key.
- Believing SNAPSHOT = SERIALIZABLE → write skew in production → add a locking read, a materialized conflict row, or move to true Serializable.
- Assuming a ledger is serializable end-to-end when its outbox commits in a separate transaction → the guarantee ends at the transaction boundary; write the outbox row in the same transaction.
- Copy-pasting `BEGIN SERIALIZABLE` between services with different contention → retry storms → review isolation per use case.

### 10. 🎯 Interview kit
**30-second answer:** Serializable guarantees results equal some serial order. It costs either blocking and deadlocks (lock-based) or tracking and serialization-failure aborts (SSI), so throughput drops and the app must retry. Use it narrowly where an invariant can't be expressed as a constraint.

**Structure a longer answer:** define serializability and the anomalies it removes (phantoms, write skew) → show how this engine implements it → quantify costs (locks held, abort rate) → alternatives (constraint, `FOR UPDATE`, optimistic version column) → a concrete scenario and the retry design.

**Likely follow-ups:**
- *Lock-based vs PostgreSQL SSI?* → locks block; SSI aborts at commit on dangerous rw cycles; SSI can false-positive.
- *Why Oracle's SERIALIZABLE isn't really serializable?* → it's snapshot isolation; write skew passes.
- *How to measure retry rate and set an SLO?* → count 40001/1205 per transaction type; alert on retry ratio, not just latency.
- *Making retries safe?* → idempotency keys, outbox pattern, no external effects before commit.

**Signals interviewers look for:** engine-specific behaviour, awareness of write skew, retry/idempotency design, and the instinct to reach for a constraint first.

### 11. Check yourself
- [ ] Draw the precedence graph of the doctors example and show the cycle.
- [ ] Explain why a missing index makes SERIALIZABLE far more blocking in SQL Server.
- [ ] State what SSI's "dangerous structure" is and why it can abort innocent transactions.
- [ ] Write a retry loop that is safe for a card charge.
- [ ] Give two ways to avoid Serializable for the on-call invariant.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [A colleague set the whole app to SERIALIZABLE "to be safe." What throughput and deadlock consequences do you warn them about?](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30147_A_colleague_set_the_whole_app_to_SERIALIZABLE_to_be_safe_Wha.md>)
- [Deep] [In Oracle you get ORA-08177 'can't serialize access' under SERIALIZABLE. What causes it and how do you design the app to handle it?](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30169_In_Oracle_you_get_ORA-08177_can_t_serialize_access_under_SER.md>)
- [Deep] [How would you measure the abort/retry rate under SERIALIZABLE in production and set an SLO around it? What signals tell you the level is too aggressive?](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30241_How_would_you_measure_the_abort_retry_rate_under_SERIALIZABL.md>)
- [Deep] [Give a step-by-step of how you'd add SERIALIZABLE only around the critical section of one endpoint while keeping the rest at READ COMMITTED. What are the gotchas?](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30278_Give_a_step-by-step_of_how_you_d_add_SERIALIZABLE_only_aroun.md>)
- [Expert] [Compare the cost of achieving serializability via two-phase locking versus via SSI in terms of blocking, abort rate, and read-only workloads.](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30206_Compare_the_cost_of_achieving_serializability_via_two-phase_.md>)
- [Expert] [Explain how PostgreSQL decides a Serializable transaction has a 'dangerous structure' (rw-antidependencies) that forces an abort. What are pivot transactions?](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30249_Explain_how_PostgreSQL_decides_a_Serializable_transaction_ha.md>)
- [Expert] [As bar-raiser probe: prove to me that your ledger is serializable end-to-end when it uses SSI in Postgres but the app-side outbox pattern commits in a separate transaction.](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30093_As_bar-raiser_probe_prove_to_me_that_your_ledger_is_serializ.md>)
- [Expert] [You're standardizing on Postgres but one team needs Oracle-style SERIALIZABLE-as-snapshot semantics for a ported app. How do you bridge the gap without rewriting their transactions?](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30130_You_re_standardizing_on_Postgres_but_one_team_needs_Oracle-s.md>)

## Related concepts

- [Range Locks and Retries](<../../Locking/Range_Locks_and_Retries.md>) *(Locking)*
- [Isolation Anomalies](<../../../SQL_Language/SQL_Fundamentals/Isolation_Anomalies.md>) *(SQL Fundamentals)*
- [Isolation Levels in Transactions](<../../ACID_Transactions/Isolation_Levels_in_Transactions.md>) *(ACID Transactions)*

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Serializability › Actual Serial Execution
- *Designing Data-Intensive Applications* — Martin Kleppmann — Serializable Snapshot Isolation (SSI)
- *Designing Data-Intensive Applications* — Martin Kleppmann — Serializable Snapshot Isolation (SSI) › Performance of serializable snapshot isolation
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 9. Consistency and Consensus › What Makes a System Linearizable?
- *Database Internals* — Alex Petrov — Transaction Processing and Recovery › Serializability
- *Database Internals* — Alex Petrov — Distributed Transactions › Distributed Transactions with Percolator
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Architecting Your Lakehouse › Serializable writes
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT
- *Think Distributed Systems* — Dominik Tornow — Transactions › 5.3.2 Serializability
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Serialization and Deserialization

