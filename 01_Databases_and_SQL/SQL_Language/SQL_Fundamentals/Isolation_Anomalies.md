---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Isolation_Anomalies"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: ["AWS", "MySQL"]
---

# Isolation Anomalies

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Isolation levels decide which concurrency anomalies (dirty read, non-repeatable read, phantom, lost update, write skew) your transaction is allowed to see; stronger levels remove anomalies by paying in locking, versioning, or retries.

### 1. The big picture
A database runs many transactions at once. *Isolation* (the "I" in ACID) is the promise that concurrent transactions do not corrupt each other's view of the data. The perfect version of that promise is **serializability**: the outcome is the same as if the transactions had run one after another. Perfect isolation is expensive, so SQL defines weaker **isolation levels** that tolerate specific, named **anomalies** in exchange for more concurrency.

This lesson is the map: *anomaly → which level prevents it → how the engine enforces it (locks vs row versions) → what you do in application code when the default level is not enough.* It sits under transactions and concurrency, and feeds directly into locking/blocking, deadlocks, and optimistic concurrency.

### 2. Core concepts and vocabulary
- **Transaction** — a unit of work bounded by `BEGIN`/`COMMIT`/`ROLLBACK`; all-or-nothing.
- **Dirty read** — reading data another transaction has changed but not yet committed. If that transaction rolls back, you acted on data that never existed.
- **Non-repeatable read** — you read a row twice in one transaction and get different values because someone committed an `UPDATE`/`DELETE` in between.
- **Phantom read** — you run the same range query twice and the *set of rows* differs because someone committed an `INSERT` (or delete) matching your predicate.
- **Lost update** — two transactions read the same value, both compute a new one, both write; the second write silently overwrites the first.
- **Write skew** — two transactions each read an overlapping set, make *different* writes based on what they read, and together violate an invariant (e.g., two doctors both go off-call because each saw the other on-call).
- **Pessimistic (lock-based) isolation** — readers take shared locks, writers take exclusive locks; conflicts cause blocking.
- **Optimistic / row-versioning (MVCC)** — writers create a new version of a row; readers see a consistent snapshot and don't block on writers. Conflicts are detected at write/commit time.
- **Snapshot** — a transaction's frozen view of committed data as of some moment.

### 3. How it works, step by step
**The four ANSI levels and what each forbids**

| Level | Dirty read | Non-repeatable read | Phantom | Typical enforcement |
|---|---|---|---|---|
| READ UNCOMMITTED | allowed | allowed | allowed | no shared locks taken (`NOLOCK`) |
| READ COMMITTED | **prevented** | allowed | allowed | short shared locks, or statement-level snapshot |
| REPEATABLE READ | prevented | **prevented** | allowed (in lock-based engines) | shared locks held to end of transaction |
| SERIALIZABLE | prevented | prevented | **prevented** | range (key-range) locks, or serializable snapshot with conflict checks |

Step by step for a lock-based engine such as SQL Server at its default **READ COMMITTED**:
1. A reader requests a shared (S) lock on each row/page it reads.
2. If a writer holds an exclusive (X) lock on an uncommitted change, the reader **waits** — that is what prevents the dirty read.
3. As soon as the row is read, the S lock is released. Nothing stops another transaction changing the row before you read it again → non-repeatable reads are possible.
4. In `REPEATABLE READ`, step 3 changes: S locks are held until commit, so rows you already read can't be modified. But rows that *don't exist yet* can still be inserted into your range → phantoms.
5. In `SERIALIZABLE`, the engine also takes **key-range locks** on the index range your predicate covers. A concurrent `INSERT` into that range blocks until you commit — so a second run of the query returns exactly the same rows.

For a **row-versioning engine** (PostgreSQL, Oracle, SQL Server with `READ_COMMITTED_SNAPSHOT` or `SNAPSHOT` isolation):
1. Every committed change leaves older versions available (a version store in `tempdb` in SQL Server, old tuples in the heap in PostgreSQL).
2. A reader picks the version that was committed when its statement (READ COMMITTED) or transaction (REPEATABLE READ / SNAPSHOT) began.
3. Readers never block writers and writers never block readers.
4. Write–write conflicts are still handled: the second writer waits for the first, and under snapshot isolation it then **fails** with an update-conflict error (SQL Server error 3960; PostgreSQL "could not serialize access") rather than overwriting.

### 4. Worked example
Two sessions against an `Inventory` table. Goal: sell one unit only if stock > 0.

```sql
-- Setup
CREATE TABLE dbo.Inventory (ProductId int PRIMARY KEY, Qty int NOT NULL);
INSERT dbo.Inventory VALUES (1, 1);

-- Session A (READ COMMITTED, default)
BEGIN TRAN;
DECLARE @q int = (SELECT Qty FROM dbo.Inventory WHERE ProductId = 1); -- sees 1
-- Session B now does exactly the same read and also sees 1
IF @q > 0
    UPDATE dbo.Inventory SET Qty = @q - 1 WHERE ProductId = 1;
COMMIT;
-- Session B then runs UPDATE ... SET Qty = 1 - 1 and commits.
-- Two orders were accepted for one unit: a LOST UPDATE / oversell.
```

Fixes, from least to most heavy:
```sql
-- 1. Atomic statement: check and write in one operation
UPDATE dbo.Inventory SET Qty = Qty - 1
WHERE ProductId = 1 AND Qty > 0;      -- check @@ROWCOUNT = 1 for success

-- 2. Take an update lock on read so the second reader waits
SELECT @q = Qty FROM dbo.Inventory WITH (UPDLOCK, HOLDLOCK) WHERE ProductId = 1;

-- 3. Run at SERIALIZABLE (or SNAPSHOT + retry on conflict)
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```
Reading the result: with fix 1 the second session's `UPDATE` waits for the first's X lock, then re-evaluates `Qty > 0` on the new value 0 and updates **zero rows** — the app sees `@@ROWCOUNT = 0` and reports "sold out".

Phantom demonstration (category filter): Connection 1 at `SERIALIZABLE` runs `SELECT ... WHERE categoryid = 1`; Connection 2's `INSERT` of a category-1 product **blocks** until Connection 1 commits, so Connection 1's second query shows no phantom. At lower levels the insert succeeds immediately and the second query returns an extra row.

### 5. Under the hood
- **Lock modes.** S (shared), X (exclusive), U (update — prevents the classic S→X conversion deadlock), IS/IX (intent locks at table/page level so the engine can cheaply tell whether a row-level lock exists below). Range locks (`RangeS-S`, `RangeX-X`, etc.) attach to index keys; this is why `SERIALIZABLE` in SQL Server is only as precise as the **index** available for the predicate — with no usable index it escalates to locking the whole table.
- **Snapshot isolation is not serializable.** It prevents dirty/non-repeatable reads and phantoms for *reads*, but permits **write skew**, because the two transactions write different rows and so never conflict. PostgreSQL's `SERIALIZABLE` is *Serializable Snapshot Isolation* (SSI): it tracks read–write dependencies and aborts one transaction when a dangerous cycle appears — hence "could not serialize" errors you must retry.
- **Where versions live.** SQL Server: `tempdb` version store (or the user database with Accelerated Database Recovery). Each modified row gets a 14-byte versioning tag, so enabling RCSI grows rows and tempdb. PostgreSQL: dead tuples remain in the heap until `VACUUM` — see the MVCC bloat lesson.
- **Default levels differ.** SQL Server and PostgreSQL default to READ COMMITTED, but SQL Server's is lock-based unless RCSI is on (it is on by default in Azure SQL Database), PostgreSQL's is statement-snapshot. MySQL InnoDB defaults to REPEATABLE READ (snapshot + gap locks). Oracle has only READ COMMITTED and SERIALIZABLE (snapshot).
- **Distributed note.** Across services or shards you rarely get a single serializable transaction; systems use two-phase commit (blocking, poor availability) or sagas with compensations plus idempotency. Coordination-avoidance techniques (e.g., RAMP) keep multiple versions around only while readers need them.

### 6. Performance and design implications
- Stronger isolation = more locks held longer = more blocking, deadlocks and, with versioning, more retries. Don't default to `SERIALIZABLE` "to be safe"; use it for narrow critical sections.
- `READ UNCOMMITTED`/`NOLOCK` is not a free speed-up: it can return rows twice or skip rows during page splits as well as dirty data.
- Under row versioning, readers stop blocking — often the single biggest relief for reporting-on-OLTP. Cost: version-store space and longer-lived versions if any transaction stays open.
- Keep transactions short; never hold a transaction open across a user prompt or network call.
- Measure: SQL Server `sys.dm_tran_locks`, `sys.dm_os_wait_stats` (`LCK_M_*` waits), `sys.dm_tran_version_store_space_usage`, blocked-process report; PostgreSQL `pg_locks`, `pg_stat_activity`, `pg_stat_database.conflicts`/`deadlocks`.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SET TRANSACTION ISOLATION LEVEL …` | Sets level for the session | Per-batch control |
| `READ_COMMITTED_SNAPSHOT ON` | READ COMMITTED uses row versions | Remove reader/writer blocking, no code change |
| `ALLOW_SNAPSHOT_ISOLATION ON` + `SNAPSHOT` | Transaction-level snapshot | Consistent multi-statement reads |
| `WITH (UPDLOCK, HOLDLOCK)` | Update lock held to commit + range protection | Check-then-write patterns |
| `SELECT … FOR UPDATE` (PG/MySQL) | Row lock on read | Same, in those engines |
| `rowversion` / version column | Optimistic concurrency check | UI edits, long think-time |
| `SET LOCK_TIMEOUT` | Fail instead of waiting forever | Protect interactive paths |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| READ COMMITTED (default) | You do check-then-act on a value (lost update, oversell) |
| RCSI / snapshot reads | Heavy tempdb pressure or long open transactions |
| REPEATABLE READ | You need phantom protection |
| SERIALIZABLE | High write concurrency on hot ranges; be ready for blocking/retries |
| Optimistic version column | Contention is high — retries thrash |
| Unique constraint as the guard | Rule isn't expressible as uniqueness |

### 9. Common mistakes
- Assuming READ COMMITTED prevents lost updates → silent overwrites → use atomic `UPDATE`, locking hints, or optimistic version check.
- Using `NOLOCK` on money paths → inconsistent or double-counted rows → use RCSI/snapshot.
- Confusing ACID "consistency" with CAP "consistency" → mixing up constraint validity with replica agreement.
- Treating snapshot as serializable → write skew slips through → add explicit locks or a constraint, or use true serializable.
- Not retrying serialization/deadlock errors → user-visible failures → wrap in bounded retry with backoff.
- Holding a transaction across an HTTP call → long locks, pool exhaustion → commit before calling out; use outbox/saga.

### 10. 🎯 Interview kit
**30-second answer:** Isolation levels trade correctness for concurrency. READ UNCOMMITTED allows dirty reads; READ COMMITTED stops those; REPEATABLE READ also stops non-repeatable reads; SERIALIZABLE also stops phantoms. Lock-based engines enforce this with held locks and key-range locks; MVCC engines with snapshots. Snapshot still permits write skew, and none of the lower levels stop lost updates by themselves.
**Structure a longer answer:** clarify the anomaly in the scenario → pick the lowest level or mechanism that prevents it → explain how the engine enforces it → state the cost (blocking vs retries) → close with a real case (inventory, seat booking).
**Likely follow-ups:**
- *Why is READ COMMITTED the common default?* → no dirty reads, low locking overhead, good enough for most OLTP.
- *How do you prevent double-booking?* → unique constraint on (resource, slot) is simplest; else `SELECT … FOR UPDATE` / `UPDLOCK`; SERIALIZABLE as the general fallback.
- *Why do I get "could not serialize access"?* → SSI/snapshot detected a conflict; retry the whole transaction.
- *Difference between a deadlock and blocking?* → blocking is waiting that ends; deadlock is a cycle the engine breaks by killing a victim.
- *Read replica vs lower isolation for analytics?* → replica or CDC isolates load; lower isolation just risks wrong data.
**Signals interviewers look for:** name anomalies precisely, distinguish lock-based from MVCC, know snapshot ≠ serializable, mention retries, tie to a concrete failure.

### 11. Check yourself
- [ ] Define dirty read, non-repeatable read, phantom, lost update and write skew with one example each.
- [ ] Fill in the anomaly matrix for the four ANSI levels from memory.
- [ ] Explain how key-range locks stop a phantom, and why it needs an index.
- [ ] Explain why snapshot isolation allows write skew but READ COMMITTED + `UPDLOCK` doesn't.
- [ ] Write an oversell-proof `UPDATE` and check `@@ROWCOUNT`.
- [ ] State what happens to a retry loop under `SERIALIZABLE` in PostgreSQL.
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)
- [MySQL](<../../../TOOLS/MySQL/_index.md>)

## Practice questions

- [Intermediate] [What is a transaction and what are the ACID properties?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40342_What_is_a_transaction_and_what_are_the_ACID_properties.md>)
- [Deep] [Explain connection-pool exhaustion under load, why serverless/lambda workloads worsen it, and how PgBouncer transaction-pooling mode changes which SQL features (temp tables, prepared statements) you can use.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39939_Explain_connection-pool_exhaustion_under_load_why_serverless.md>)
- [Deep] [What is the READ COMMITTED isolation level's behavior, and why is it the common default?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40255_What_is_the_READ_COMMITTED_isolation_level_s_behavior_and_wh.md>)
- [Deep] [What is a deadlock in a database and how can you prevent it?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40387_What_is_a_deadlock_in_a_database_and_how_can_you_prevent_it.md>)
- [Deep] [You must guarantee a global unique invoice number with no gaps for compliance, in a high-concurrency system. Why do sequences/auto-increment leave gaps and how would you design a gapless counter safely?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40292_You_must_guarantee_a_global_unique_invoice_number_with_no_ga.md>)
- [Deep] [Your analytics team wants ad-hoc queries against production and it's causing lock contention and IO spikes. What isolation, replica, or CDC-based approach would you propose and why?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40321_Your_analytics_team_wants_ad-hoc_queries_against_production_.md>)
- [Expert] [How do transactions interact with connection pooling and auto-commit in a backend app?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39811_How_do_transactions_interact_with_connection_pooling_and_aut.md>)
- [Expert] [How do long-running transactions hurt a database, and what is the risk of holding a transaction open across a network call?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40261_How_do_long-running_transactions_hurt_a_database_and_what_is.md>)

## Related concepts

- [Distributed Transactions and 2PC](<../../Database_Internals/DBMS_Concepts/Distributed_Transactions_and_2PC.md>) *(DBMS Concepts)*
- [Lock Holding and Optimistic Concurrency](<../../Transactions_and_Concurrency/ACID_Transactions/Lock_Holding_and_Optimistic_Concurrency.md>) *(ACID Transactions)*
- [Range Locks and Retries](<../../Transactions_and_Concurrency/Locking/Range_Locks_and_Retries.md>) *(Locking)*

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Weak Isolation Levels
- *Designing Data-Intensive Applications* — Martin Kleppmann — Write Skew and Phantoms › Materializing conflicts
- *Designing Data-Intensive Applications* — Martin Kleppmann — Aiming for Correctness › Duplicate suppression
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Transactions and concurrency
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Infrastructure Engineering › Concurrency

