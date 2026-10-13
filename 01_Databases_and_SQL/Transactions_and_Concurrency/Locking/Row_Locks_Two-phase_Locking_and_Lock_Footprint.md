---
id: "01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Row_Locks_Two-phase_Locking_and_Lock_Footprint"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Transactions_and_Concurrency/Locking"
tools: ["MySQL"]
---

# Row Locks Two-phase Locking and Lock Footprint

**L4 Advanced** · [↑ Locking](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Row locks, held under two-phase locking until commit, make concurrent writers safe; the number and granularity of locks a statement acquires — its lock footprint — decides how much it blocks others and when the engine escalates to a table lock.

### 1. The big picture
A lock is a logical claim on a piece of data, taken on behalf of a transaction and kept in the database's lock manager. Locks are distinct from **latches**, which are short-lived internal mutexes that protect in-memory structures (a B-tree page while it is being split) for microseconds. Locks protect *logical* consistency between transactions and live for the duration of the transaction; latches protect *physical* integrity and live for the duration of a single operation.

Three ideas fit together:

1. **Granularity** — locks can be taken at row, page, table, or database level. Finer granularity gives more concurrency, but each lock costs memory and CPU.
2. **Two-phase locking (2PL)** — a transaction first only *acquires* locks (growing phase), then only *releases* them (shrinking phase). Strict 2PL holds all exclusive locks until commit, which is what makes histories serializable and recoverable.
3. **Lock footprint** — how many locks, of which type and level, a statement takes. A statement that touches 10 rows holds 10 row locks; one touching 6 million rows can't hold 6 million row locks cheaply, so the engine *escalates*.

### 2. Core concepts and vocabulary
- **Lock mode** — S (shared/read), X (exclusive/write), U (update), and **intent** modes IS, IX, SIX that announce "I hold finer-grained locks below this node".
- **Compatibility matrix** — which modes can be held together on the same resource.
- **Lock hierarchy** — database → table (object) → partition → page → row/key.
- **Lock escalation** — converting many fine locks into one coarser lock to save memory.
- **Lock footprint** — the set of locks a statement acquires; drives memory use and blocking radius.
- **Blocking** — one session waits for a lock another holds (normal, temporary).
- **Deadlock** — circular wait; engine kills one transaction.
- **Lock duration** — Read Committed releases S locks right after reading; Repeatable Read keeps them to commit; X locks are always kept to commit.
- **Transaction-ID (TID) lock** — in optimized locking (SQL Server 2025/Azure SQL), a single lock on the writing transaction's ID replaces thousands of row/page locks.

### 3. How it works, step by step
1. **Statement arrives**; the optimizer picks an access path (index seek vs scan). The path determines which keys will be touched.
2. **Intent locks first.** Before locking a row, the engine takes IX on the table and page. A later `ALTER TABLE` needing a table-level X lock can see the IX and wait without scanning rows.
3. **Row/key lock.** For each row to modify: request X (or U then convert to X). If another transaction holds an incompatible lock, wait in the queue.
4. **Modify and log.** The change is applied and logged; the lock stays.
5. **Escalation check.** If a single statement holds roughly 5,000 locks on one object (SQL Server), or lock memory exceeds ~24 % of the buffer (or 40 % of memory) the engine attempts to replace them with one table (or partition) lock. If another transaction's incompatible intent lock exists, the attempt fails and is retried after every further ~1,250 locks.
6. **Commit.** All locks are released together (strict 2PL).

Compatibility (simplified):

| Held \ Requested | IS | IX | S | U | X |
|---|---|---|---|---|---|
| IS | ✓ | ✓ | ✓ | ✓ | ✗ |
| IX | ✓ | ✓ | ✗ | ✗ | ✗ |
| S | ✓ | ✗ | ✓ | ✓ | ✗ |
| U | ✓ | ✗ | ✓ | ✗ | ✗ |
| X | ✗ | ✗ | ✗ | ✗ | ✗ |

### 4. Worked example
Escalation from a big update:

```sql
BEGIN TRAN;
UPDATE Sales.SalesOrderDetail
SET    ProductID = ProductID
WHERE  ProductID <= 712;           -- ~ tens of thousands of rows

SELECT resource_type, request_mode, request_status, COUNT(*) AS n
FROM   sys.dm_tran_locks
WHERE  request_session_id = @@SPID
GROUP  BY resource_type, request_mode, request_status;
```

You will typically see a single `OBJECT` lock in `X` mode instead of thousands of `KEY`/`PAGE` locks. To confirm the escalation happened:

```sql
SELECT OBJECT_NAME(object_id) AS tbl, index_id,
       index_lock_promotion_attempt_count AS attempts,
       index_lock_promotion_count         AS succeeded
FROM   sys.dm_db_index_operational_stats(DB_ID(), NULL, NULL, NULL);
```

`attempts` > 0 with `succeeded` > 0 means the table was locked. From that moment, every other session that touches the table waits — even for unrelated rows. Fixes: batch the update (`WHILE` loop, 1,000–4,000 rows), add an index so the statement reads fewer rows, or set `ALTER TABLE … SET (LOCK_ESCALATION = AUTO)` for partitioned tables so escalation goes to the partition.

### 5. Under the hood
- **Cost per lock.** A SQL Server lock is about 96 bytes of lock-manager memory; 100,000 row locks ≈ 10 MB. That's why escalation exists.
- **Where locks live.** SQL Server and InnoDB keep locks in lock-manager hash tables outside the data pages. PostgreSQL stores row-lock information *in the tuple header* (`xmax` plus infomask bits), so a row lock can mean a disk write even for `SELECT … FOR UPDATE`, and has no escalation — instead it limits the shared lock table to table-level locks.
- **Optimized locking (SQL Server 2025 / Azure).** Requires row versioning/ADR. Writers take a single X lock on their *transaction ID*; row/page locks are acquired and released immediately. Combined with *lock after qualification (LAQ)* — evaluating the predicate on the latest committed version without locking, then locking only matching rows — blocking and escalation largely disappear.
- **Latches** (`PAGELATCH_*`, `PAGEIOLATCH_*`) show up when many sessions insert into the last page of an index; that is *not* lock blocking and the fix is different (partitioning, hash keys, `OPTIMIZE_FOR_SEQUENTIAL_KEY`).
- **Index effect.** The more rows a query reads before filtering, the more locks it takes (under higher isolation) or the longer it waits behind others (even under RC, S locks must be *acquired*). A missing index multiplies footprint.
- **Waits vs locks.** `LCK_M_X` (waiting for exclusive), `LCK_M_S`, `LCK_M_U`, `LCK_M_IX` in `sys.dm_os_wait_stats` tell you which mode is the bottleneck.

### 6. Performance and design implications
- Keep transactions short; no user input while holding locks; touch objects in a consistent order.
- Use the lowest isolation level that satisfies correctness; consider RCSI to eliminate reader S-lock waits.
- Index to make predicates seek, which shrinks both the footprint and the chance of conflicts.
- Batch large modifications to stay under escalation thresholds and keep the log small.
- For work queues use `FOR UPDATE SKIP LOCKED` (PostgreSQL/MySQL 8) or `READPAST` + `UPDLOCK, ROWLOCK` (SQL Server) so consumers don't line up behind one row.
- Observe: `sys.dm_tran_locks`, `sys.dm_os_waiting_tasks`, blocked-process report (extended event), `log_lock_waits` in PostgreSQL, `performance_schema.data_locks` in MySQL.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `sys.dm_tran_locks` | Lists granted/waiting locks | Who holds what |
| `sp_whoisactive` / `sys.dm_exec_requests.blocking_session_id` | Find the head blocker | Live blocking |
| `ALTER TABLE … SET (LOCK_ESCALATION = TABLE\|AUTO\|DISABLE)` | Controls escalation target | Partitioned tables, queue tables |
| `WITH (ROWLOCK \| PAGLOCK \| TABLOCK \| UPDLOCK \| READPAST)` | Per-statement lock hints | Targeted control |
| `SET LOCK_TIMEOUT` / `lock_timeout` / `NOWAIT` | Limit waiting | Interactive paths |
| `ALTER DATABASE … SET OPTIMIZED_LOCKING = ON` | Enable TID locking | SQL Server 2025 / ADR databases |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Row-level locks for OLTP | Bulk loads where one table lock is cheaper (`TABLOCK`) |
| Allow escalation (default) | Escalation causes outages on shared hot tables |
| Higher isolation for correctness | Locks held for long durations on busy keys |
| `NOWAIT` / `SKIP LOCKED` | You must process every row in strict order |

### 9. Common mistakes
- Updating millions of rows in one statement → escalation to a table X lock → batch it.
- Disabling escalation globally → lock manager memory pressure → target specific tables.
- Treating `NOLOCK` as a performance tool → dirty/duplicate/missing rows → use row versioning.
- Blaming locks for `PAGELATCH` contention → wrong fix → read wait types first.
- Inconsistent update order across code paths → deadlocks → document and enforce one order.
- Holding a transaction open across a network call → everyone queues → commit before the call.

### 10. 🎯 Interview kit
**30-second answer:** Writers take exclusive row locks and hold them to commit (two-phase locking), with intent locks above them so table-level requests can be resolved quickly. Many row locks escalate to a table lock to save memory, which widens blocking, so you keep transactions short, index well and batch large changes.

**Structure a longer answer:** lock vs latch → modes and compatibility → 2PL/strict 2PL → granularity and escalation → how to observe → how to design around it.

**Likely follow-ups:**
- *Why prefer row locks over table locks?* → concurrency; cost is memory per lock.
- *What is lock escalation and how do you prevent it?* → many locks → one; batching, indexing, partition-level escalation.
- *Lock vs latch?* → logical, transaction-duration vs physical, instruction-duration.
- *How do you find the blocker?* → head of the blocking chain via `blocking_session_id`.

**Signals interviewers look for:** correct vocabulary, you can name DMVs, and you connect footprint to index design and batch size.

### 11. Check yourself
- [ ] Fill in the S/U/X/IX compatibility matrix from memory.
- [ ] Explain why intent locks exist.
- [ ] State two thresholds that trigger escalation and how to see it happened.
- [ ] Give three ways to reduce a statement's lock footprint.
- [ ] Describe how optimized locking changes what a writer locks.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../../../TOOLS/MySQL/_index.md>)

## Practice questions

- [Basic] [In one sentence, what does it mean for two operations to be concurrent in a database?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Q-31293_In_one_sentence_what_does_it_mean_for_two_operations_to_be_c.md>)
- [Intermediate] [What does the SELECT ... FOR UPDATE statement do?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Q-31286_What_does_the_SELECT_FOR_UPDATE_statement_do.md>)
- [Intermediate] [What is the difference between blocking and deadlock?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Q-31347_What_is_the_difference_between_blocking_and_deadlock.md>)
- [Deep] [What is a next-key lock in InnoDB and how is it related to gap locks?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Q-31307_What_is_a_next-key_lock_in_InnoDB_and_how_is_it_related_to_g.md>)
- [Deep] [How does locking behave differently in an OLTP system versus a reporting query?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Q-31370_How_does_locking_behave_differently_in_an_OLTP_system_versus.md>)
- [Deep] [Your service retries on deadlock but the retry storm makes contention worse. How would you tune backoff and batch size to break the cycle?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Q-31218_Your_service_retries_on_deadlock_but_the_retry_storm_makes_c.md>)
- [Deep] [How would you architect a globally unique sequence generator that avoids the single-row lock bottleneck of a naive counter table?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Q-31255_How_would_you_architect_a_globally_unique_sequence_generator.md>)
- [Deep] [A nightly reconciliation job and live traffic contend on the same rows. How would you schedule or isolate the job to remove the contention without stale results?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Q-31291_A_nightly_reconciliation_job_and_live_traffic_contend_on_the.md>)

## Related concepts

- [Lock Holding and Optimistic Concurrency](<../ACID_Transactions/Lock_Holding_and_Optimistic_Concurrency.md>) *(ACID Transactions)*
- [SKIP LOCKED Job Queues](<../../SQL_Language/SQL_Fundamentals/SKIP_LOCKED_Job_Queues.md>) *(SQL Fundamentals)*
- [Cross-service Invalidation Races](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Cross-service_Invalidation_Races.md>) *(Concurrency Patterns)*
- [Deadlock Causes and Diagnosis](<../Deadlocks/Deadlock_Causes_and_Diagnosis.md>) *(Deadlocks)*
- [Deadlocks in MySQL](<../../../TOOLS/MySQL/MySQL_Platform/Deadlocks_in_MySQL.md>) *(MySQL Platform)*
- [Lock Granularity](<../../Database_Internals/DBMS_Concepts/Lock_Granularity.md>) *(DBMS Concepts)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Lock Granularity
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Lock Escalation
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Best Practices
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Hands-on Labs
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 16 Blocking And Blocked Processes
- *Database Internals* — Alex Petrov — Transaction Processing and Recovery › Pessimistic Concurrency Control
- *Database Internals* — Alex Petrov — Transaction Processing and Recovery › Lock-Based Concurrency Control
- *Database Internals* — Alex Petrov — Transaction Processing and Recovery › Locks
- *Database Internals* — Alex Petrov — Transaction Processing and Recovery › Latches
- *Designing Data-Intensive Applications* — Martin Kleppmann — Preventing Lost Updates › Explicit locking
- *Designing Data-Intensive Applications* — Martin Kleppmann — Two-Phase Locking (2PL) › Performance of two-phase locking
- *Designing Data-Intensive Applications* — Martin Kleppmann — Don't confuse 2PC and 2PL › Holding locks while in doubt
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Locking Contention
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Lock Escalation
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Latch Contention
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page Latch
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Operational Visibility › Locking and Concurrency

