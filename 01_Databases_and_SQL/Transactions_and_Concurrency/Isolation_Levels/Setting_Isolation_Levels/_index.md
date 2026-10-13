---
id: "01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels"
tools: []
---

# Setting Isolation Levels

**L4 Advanced** · [↑ Isolation Levels](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Choosing an isolation level is a per-workload decision about how much anomaly risk you accept versus blocking and aborts. **Where** you set it (database, session, transaction or pool) decides whether it actually applies.

### 1. What it is
Every transaction runs at some isolation level, and that level comes from several layers:
1. **Database default.** SQL Server: READ COMMITTED (with RCSI on by default in Azure SQL). PostgreSQL and Oracle: READ COMMITTED. MySQL InnoDB: REPEATABLE READ.
2. **Database options** that change what a level *means*. In SQL Server, `READ_COMMITTED_SNAPSHOT ON` turns READ COMMITTED into statement-level versioning, and `ALLOW_SNAPSHOT_ISOLATION ON` lets sessions use SNAPSHOT.
3. **Session / transaction.** `SET TRANSACTION ISOLATION LEVEL …`. In SQL Server this lasts for the session; in PostgreSQL it applies to the transaction.
4. **Client / driver / pool.** JDBC `setTransactionIsolation`, ORM settings, and pool defaults that may **reset or leak** a level between borrowed connections.
5. **Statement hints.** For example `WITH (NOLOCK)`, `READCOMMITTEDLOCK`, `FOR UPDATE`.

### 2. Why it matters
- The same code can behave differently in production because a pooled connection kept a previous session's level.
- Too low a level causes silent anomalies; too high causes blocking, deadlocks and serialization failures.
- Different workloads on one database (checkout vs dashboards) need different levels.

### 3. How it works: choosing per workload
| Workload | Sensible choice | Reason |
|---|---|---|
| Dashboards, analytics reads | Snapshot / RCSI | No blocking on writers, consistent snapshot |
| Typical OLTP CRUD | READ COMMITTED (versioned if available) + constraints | Good concurrency; constraints enforce the invariants |
| Check-then-act invariants (bookings, balances) | Targeted `FOR UPDATE` / anchor locks, or SERIALIZABLE on that path only | Protect only what needs it |
| Job queues | READ COMMITTED + `SELECT … FOR UPDATE SKIP LOCKED` | Workers grab different rows without blocking each other |
| Rough counters, metrics | READ COMMITTED; accept small drift | Correctness isn't worth contention |
| READ UNCOMMITTED | Almost never; maybe a rough monitoring query on a hot table | Risk of dirty, missing or duplicated rows |

### 4. Key details to remember
- In SQL Server, setting the level **after** the first statement of a transaction does not retroactively change what was already read. Set it before `BEGIN TRAN`.
- SQL Server's SNAPSHOT needs the database option **and** a session `SET`. RCSI needs only the database option (and exclusive access to switch it on).
- Row-versioning levels move cost to **tempdb** (SQL Server) or **undo / dead tuples** (Oracle / PostgreSQL), so long transactions cause version-store growth or bloat.
- At any level stricter than READ COMMITTED, the application must **retry** on deadlock or serialization errors.

### 5. Trade-offs
| Higher isolation gives you | It costs you |
|---|---|
| Fewer anomalies, simpler reasoning | Blocking (locking engines) or aborts (MVCC engines) |
| A correct check-then-act | Retry logic and idempotency everywhere |
| Consistent multi-statement reads | Version store / undo growth during long transactions |

### 6. Common mistakes
- Setting the level in code without checking the **pool default** → inconsistent behaviour → set it explicitly per transaction and reset it on return to the pool.
- Raising the whole database's level after one bug → system-wide latency → scope the fix to the critical path.
- Using `NOLOCK` as a performance fix → wrong results → enable RCSI instead.
- Long-running snapshot reports → tempdb or VACUUM pressure → keep transactions short or read from a replica.

### 7. 🎯 Interview kit
**30-second answer:** "I pick isolation per workload. Reads and dashboards use snapshot-based levels so they never block writers. Normal OLTP stays at READ COMMITTED with constraints enforcing the invariants. The few check-then-act paths get targeted row locks or SERIALIZABLE with a retry loop. I also set the level explicitly per transaction, because pool and driver defaults can silently differ from the database default."

**Structure a longer answer:**
1. Clarify: engine, current defaults, the invariants at risk, read/write mix and latency SLOs.
2. Core idea: map each workload to a level (use the table above).
3. Trade-offs: blocking vs aborts, version-store cost, retry requirements.
4. Close: name the monitoring you'd add (deadlocks, serialization failures, tempdb/bloat, lock waits).

**Likely follow-ups:**
- *Multi-tenant SaaS, and one tenant's long transaction hurts the others. What now?* → Use versioned reads, statement timeouts, keep long jobs off the primary, and add per-tenant connection limits.
- *Which isolation level for a SKIP LOCKED job queue?* → READ COMMITTED is enough, because the row lock does the coordination.
- *Moving from a single MySQL instance at RR to shards: what changes?* → No cross-shard isolation guarantees. You need sagas or 2PC, idempotency, and per-shard invariants.
- *Defend accepting write skew in analytics counters.* → Quantify the error bound, show reconciliation, and compare against the contention cost.

**Signals interviewers look for:** workload-driven choices, awareness of the driver/pool layer, and pairing strict levels with retries and monitoring.

### 8. Check yourself
- [ ] Name the default isolation level of SQL Server, PostgreSQL, MySQL and Oracle.
- [ ] Explain the difference between enabling RCSI and enabling SNAPSHOT in SQL Server.
- [ ] Choose levels for checkout, analytics and a job queue on the same database.

*See the References section of this note for the books this lesson draws on.*
<!-- explainer:end -->

## Go deeper

- [Serializable Performance](<Serializable_Performance.md>)
- [Isolation with Read Replicas](<Isolation_with_Read_Replicas.md>)
- [Isolation for Shared Subsystems](<Isolation_for_Shared_Subsystems.md>)
- [From Serializable to Sagas](<From_Serializable_to_Sagas.md>)

## Practice questions

- [Basic] [When is READ UNCOMMITTED ever acceptable? Give a realistic use case and explain the risk you accept.](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30081_When_is_READ_UNCOMMITTED_ever_acceptable_Give_a_realistic_us.md>)
- [Intermediate] [Explain what happens to visibility if you SET TRANSACTION ISOLATION LEVEL after the first query has already run in PostgreSQL.](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30201_Explain_what_happens_to_visibility_if_you_SET_TRANSACTION_IS.md>)
- [Deep] [How do isolation levels interact with SELECT ... FOR SHARE versus FOR UPDATE? When would FOR SHARE still let a write skew slip through?](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30196_How_do_isolation_levels_interact_with_SELECT_FOR_SHARE_versu.md>)
- [Deep] [In a microservice each with its own transaction, how do you reason about isolation across service boundaries? Why doesn't a single DB isolation level save you?](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30218_In_a_microservice_each_with_its_own_transaction_how_do_you_r.md>)
- [Deep] [How do you decide whether to solve a concurrency bug with a higher isolation level, an explicit lock, a unique constraint, or an application-level mutex?](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30238_How_do_you_decide_whether_to_solve_a_concurrency_bug_with_a_.md>)
- [Deep] [How would you set the isolation level for a queue consumer that must process each message exactly once with SELECT FOR UPDATE SKIP LOCKED plus a delete?](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30264_How_would_you_set_the_isolation_level_for_a_queue_consumer_t.md>)
- [Deep] [You must set an isolation policy for a system where some transactions are safe to retry and some are not (external side effects). How do you architect the boundary and enforce it?](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30118_You_must_set_an_isolation_policy_for_a_system_where_some_tra.md>)
- [Expert] [How do you audit a large existing codebase to find every transaction whose correctness silently depends on the current default isolation level? What tooling would you build?](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30120_How_do_you_audit_a_large_existing_codebase_to_find_every_tra.md>)

## Related concepts

- [Range Locks and Retries](<../../Locking/Range_Locks_and_Retries.md>) *(Locking)*
- [Isolation Levels in Transactions](<../../ACID_Transactions/Isolation_Levels_in_Transactions.md>) *(ACID Transactions)*
- [Isolation Anomalies](<../../../SQL_Language/SQL_Fundamentals/Isolation_Anomalies.md>) *(SQL Fundamentals)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Isolation
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Combining Locks with Isolation Levels
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Isolation Levels in SQL Server
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 16: Best Practices for T-SQL Development and Database Design › Concurrency Control and Isolation Levels
- *Designing Data-Intensive Applications* — Martin Kleppmann — Weak Isolation Levels › Read Committed
- *Designing Data-Intensive Applications* — Martin Kleppmann — Serializability
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Transactions and concurrency
- *Database Internals* — Alex Petrov — Transaction Processing and Recovery › Transaction Isolation
- *Database Internals* — Alex Petrov — Transaction Processing and Recovery › Isolation Levels

