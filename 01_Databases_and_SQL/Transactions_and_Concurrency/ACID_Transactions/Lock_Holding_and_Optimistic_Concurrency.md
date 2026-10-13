---
id: "01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Lock_Holding_and_Optimistic_Concurrency"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions"
tools: ["MySQL"]
---

# Lock Holding and Optimistic Concurrency

**L4 Advanced** · [↑ ACID Transactions](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Pessimistic concurrency locks data while you work on it; optimistic concurrency lets everyone proceed and checks for conflict at write time, so the real design question is how long locks are held and who retries.

### 1. The big picture
Concurrency control answers "what if two people change the same thing?" Two philosophies exist. **Pessimistic**: assume conflict, take locks first, make others wait. **Optimistic**: assume conflict is rare, read without locking, and when writing, verify nothing changed (version column, timestamp) and fail/retry if it did.

The danger in pessimistic designs is not the lock; it is the lock's *duration*. A transaction held open across a user's coffee break, a network call, a lazy-loaded ORM relationship or a leaked connection holds locks for seconds or minutes and queues everyone behind it. Optimistic control moves the cost from waiting to retrying, which is usually better where conflicts are rare and transactions touch data across user think-time (edit forms, web UIs).

### 2. Core concepts and vocabulary
- **Lock duration** — how long a lock is held; under default isolation, write locks last until commit.
- **Blocking** — one session waiting for a lock another holds (normal, temporary).
- **Lost update** — A and B read the same value, both write; one change vanishes.
- **Pessimistic locking** — `SELECT ... WITH (UPDLOCK)` / `FOR UPDATE`.
- **Optimistic concurrency control (OCC)** — validate at write time using a version.
- **Version column / `rowversion`** — a value that changes on every update (`rowversion` in SQL Server, an integer you increment, or a timestamp).
- **Compare-and-swap update** — `UPDATE ... WHERE id = @id AND version = @v`.
- **Lock escalation** — many fine locks converted to a table lock.
- **Connection pool** — reused connections; an unclosed transaction pins the connection and its locks.
- **Dual write** — updating two systems (database and cache/index) without a shared transaction.

### 3. How it works, step by step
Optimistic update with a version column:

1. Read the row, including `RowVer`, without holding a lock (the transaction ends or is short).
2. User or code modifies the data (can take minutes; no locks held).
3. Write: `UPDATE T SET ..., Name = @n WHERE Id = @id AND RowVer = @readVer`.
4. Check rows affected: **1** means success (the engine bumps the version); **0** means someone else changed or deleted the row.
5. On 0: reload, show the conflict or merge, or retry automatically if the operation is commutative.

Pessimistic alternative:

1. `BEGIN TRAN; SELECT ... WITH (UPDLOCK, ROWLOCK) WHERE Id = @id;` — others reading for update wait.
2. Compute and `UPDATE`.
3. `COMMIT` releases locks.
This is correct and simple, but only as long as steps 1-3 are short and in one request.

Keeping lock time short:

1. Do all reads and validation **before** `BEGIN TRAN` where safe.
2. Never wait for user input, HTTP calls or message-queue acks inside a transaction.
3. Fetch needed related rows up front (joins, eager loading) instead of lazy-loading inside the transaction (N+1 queries each extending lock time).
4. Commit, then call external systems; use an outbox table to publish events reliably.

### 4. Worked example
```sql
CREATE TABLE Product (
    ProductID int PRIMARY KEY,
    Name      nvarchar(100) NOT NULL,
    Price     decimal(10,2) NOT NULL,
    RowVer    rowversion                      -- changes automatically
);

-- App reads
SELECT ProductID, Name, Price, RowVer FROM Product WHERE ProductID = 5;
-- ... user edits for 3 minutes, no locks held ...

-- App saves
UPDATE Product
   SET Price = @newPrice
 WHERE ProductID = 5 AND RowVer = @rowVerFromRead;

IF @@ROWCOUNT = 0
    -- conflict: another user changed or deleted it; reload and ask
    THROW 50010, 'The record was changed by another user.', 1;
```

Pessimistic version for a short critical section:
```sql
BEGIN TRAN;
SELECT @qty = Qty FROM Inventory WITH (UPDLOCK, ROWLOCK) WHERE ProductID = 5;
IF @qty >= @want
    UPDATE Inventory SET Qty = Qty - @want WHERE ProductID = 5;
COMMIT;
```
The single-statement form `UPDATE Inventory SET Qty = Qty - @want WHERE ProductID = 5 AND Qty >= @want` needs no explicit lock and is the best of both: atomic and short.

Duplicate-key handling is optimistic too: insert and catch the unique-violation error instead of "SELECT then INSERT", which races.

### 5. Under the hood
- `rowversion` is an 8-byte database-wide counter stamped on each modified row, so comparing it is one binary compare in the same index seek used to find the row.
- The `UPDATE ... WHERE RowVer = @v` still takes an X lock on the row while it runs, so optimistic control reduces the *window* of locking to the statement, not to zero.
- Lock modes: S (shared read), U (update, intent to write; only one holder, preventing the S→X conversion deadlock), X (exclusive). `UPDLOCK` takes U at read time so two read-modify-writers queue instead of deadlocking.
- Escalation from row to table locks happens after about 5,000 locks on one object in a statement; a bulk update inside a long transaction can therefore lock the whole table.
- Open transactions on pooled connections: if code returns a connection to the pool without commit/rollback, the next borrower inherits the open transaction and its locks (or the pool resets it, depending on driver); leaked transactions show up as long-running sessions with `open_transaction_count > 0` and sleeping status.

### 6. Performance and design implications
- Lock hold time, not lock count, is the throughput killer; measure with `sys.dm_tran_locks`, `sys.dm_exec_requests.blocking_session_id`, and wait stats `LCK_M_*`.
- OCC scales well when conflict probability is low; at high contention (a hot counter) it produces retry storms, and pessimistic or atomic-increment is better.
- Retrying costs repeating the work; keep the retried unit small and idempotent.
- ORM lazy loading inside a transaction can multiply queries and extend lock life; load aggregates eagerly before the write phase.
- NOLOCK reports avoid blocking but can double-count or skip rows; prefer RCSI.
- Detect leaked transactions with `DBCC OPENTRAN`, `sys.dm_tran_session_transactions`, and pool/`using` discipline (try/finally with commit-or-rollback).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `rowversion` column | Auto-changing version stamp | Optimistic checks |
| `WHERE ... AND RowVer = @v` + `@@ROWCOUNT` | Compare-and-swap update | Edit forms, APIs (ETag/If-Match) |
| `WITH (UPDLOCK, ROWLOCK)` | Pessimistic read-for-update | Short critical sections |
| `SET LOCK_TIMEOUT ms` | Bound waiting | Avoid indefinite blocks |
| `sys.dm_tran_locks` | Current locks | Diagnosing blocking |
| `DBCC OPENTRAN` | Oldest open transaction | Finding leaks |
| `SET DEADLOCK_PRIORITY` | Victim preference | Protect important work |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Optimistic (version check) | Conflicts frequent or retries expensive |
| Pessimistic `UPDLOCK` | The critical section includes slow external work |
| Single atomic `UPDATE ... WHERE cond` | Logic needs multi-row validation |
| Retry on conflict | Operation not idempotent |
| Snapshot isolation | Write skew risk; use constraints/locks for invariants |
| Application-level locks | Cross-system coordination you can't do otherwise (advisory locks) |

### 9. Common mistakes
- Transaction open while waiting for user input → long blocking chain → read, release, write with version check.
- Lazy-loading related entities inside a transaction → N+1 queries and extended locks → eager load first.
- Ignoring `@@ROWCOUNT = 0` after versioned update → silent lost update → treat as conflict.
- Leaked transactions under load → connection pool exhausted → `try/finally`, timeouts, monitoring of open transactions.
- "Check then insert" without constraint → duplicates under race → unique constraint and catch the violation.
- Dual write to database and cache without a transaction → divergence → write DB then publish from an outbox or use change data capture.

### 10. 🎯 Interview kit
**30-second answer:** Pessimistic control locks rows before changing them; optimistic control reads without locks and checks a version at write time, failing if it changed. Optimistic suits web apps with rare conflicts and long think time; pessimistic suits short, hot critical sections. Either way keep transactions short.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.
Define the lost update, show the version-column pattern and the `UPDLOCK` pattern, discuss retry vs wait, then the transaction-boundary hygiene.

**Likely follow-ups:**
- *Implement optimistic locking?* → version column, conditional update, rows affected 0 → conflict.
- *Why not hold a transaction during user input?* → locks pile up, blockers, log growth.
- *Lost update fix: retries or locks?* → depends on contention and idempotency; atomic update when possible.
- *Pool exhaustion from leaked transactions?* → detect via open-transaction DMVs, enforce scoping.
- *Dual write problem?* → no atomicity across systems; use outbox/CDC.

**Signals interviewers look for:** you reason about lock duration, you know the conditional update idiom, and you treat conflict as a normal outcome to handle.

### 11. Check yourself
- [ ] Write a conditional update with `rowversion` and handle conflict.
- [ ] Explain why `UPDLOCK` avoids a conversion deadlock.
- [ ] List three ways to shorten lock hold time.
- [ ] Describe how to find a leaked open transaction.
- [ ] Choose optimistic or pessimistic for three scenarios.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../../../TOOLS/MySQL/_index.md>)

## Practice questions

- [Intermediate] [What is a shared lock versus an exclusive lock?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00436_What_is_a_shared_lock_versus_an_exclusive_lock.md>)
- [Intermediate] [What happens to open transactions when a database connection is unexpectedly dropped?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00483_What_happens_to_open_transactions_when_a_database_connection.md>)
- [Intermediate] [Explain how a UNIQUE index behaves under concurrent inserts of the same key and what error your app should catch.](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00363_Explain_how_a_UNIQUE_index_behaves_under_concurrent_inserts_.md>)
- [Deep] [What is the difference between a lock and a latch in a database?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00434_What_is_the_difference_between_a_lock_and_a_latch_in_a_datab.md>)
- [Deep] [What is the lost update problem in concurrent transactions?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00462_What_is_the_lost_update_problem_in_concurrent_transactions.md>)
- [Deep] [A long-running reporting transaction is bloating your PostgreSQL tables; how does MVCC cause that and how do you mitigate it?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00320_A_long-running_reporting_transaction_is_bloating_your_Postgr.md>)
- [Deep] [How do foreign key checks acquire locks and cause unexpected contention on the parent table during inserts?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00354_How_do_foreign_key_checks_acquire_locks_and_cause_unexpected.md>)
- [Deep] [Explain how ORM-managed transactions (like session-per-request) can hide subtle isolation and flush-ordering bugs.](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00395_Explain_how_ORM-managed_transactions_like_session-per-reques.md>)

## Related concepts

- [Isolation Anomalies](<../../SQL_Language/SQL_Fundamentals/Isolation_Anomalies.md>) *(SQL Fundamentals)*
- [Deadlock Causes and Diagnosis](<../Deadlocks/Deadlock_Causes_and_Diagnosis.md>) *(Deadlocks)*
- [Row Locks Two-phase Locking and Lock Footprint](<../Locking/Row_Locks_Two-phase_Locking_and_Lock_Footprint.md>) *(Locking)*
- [Lock Granularity](<../../Database_Internals/DBMS_Concepts/Lock_Granularity.md>) *(DBMS Concepts)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Introduction
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 16: Best Practices for T-SQL Development and Database Design › Transaction and Concurrency Management
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 16: Best Practices for T-SQL Development and Database Design › Optimistic versus Pessimistic Concurrency
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Transactions and concurrency
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — has emerged as the key technology for orchestrating the deployment and maintenance of containers
- *Database Internals* — Alex Petrov — Transaction Processing and Recovery › Optimistic Concurrency Control

