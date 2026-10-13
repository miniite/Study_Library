---
id: "01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/ACID_Durability_and_Consistency"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions"
tools: []
---

# ACID Durability and Consistency

**L4 Advanced** · [↑ ACID Transactions](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** ACID is the contract that a transaction is all-or-nothing, leaves the data valid, does not trip over concurrent work, and once acknowledged survives a crash; durability and consistency are the two letters people most often misread.

### 1. The big picture
A transaction is a unit of work: several reads and writes that must behave as one. ACID names four guarantees the engine offers for that unit: **Atomicity** (all or nothing), **Consistency** (valid state to valid state), **Isolation** (concurrent transactions behave as if run in some order), **Durability** (committed means permanent). The books describe it as a contract: you promise to define logical units of work correctly, and the engine promises to handle failures, concurrency and recovery so the data stays trustworthy.

This lesson focuses on durability and consistency, because they are the subtle ones. Durability is the engine's promise about crashes and power loss; it is implemented with a write-ahead log. Consistency is two different things that share a word: the database's *C* (constraints hold at commit) and the distributed-systems *consistency* (a read returns the latest write). ACID is not mandatory for every application; many distributed stores relax it (eventual consistency) for scale, and engineers must know which model they are operating under.

### 2. Core concepts and vocabulary
- **Transaction** — a sequence of operations treated as one logical unit.
- **Atomicity** — either every change is applied or none is.
- **Consistency (ACID C)** — a transaction takes the database from one state that satisfies all declared rules to another; the engine enforces constraints, you define the business rules.
- **Isolation** — the degree to which concurrent transactions are hidden from each other.
- **Durability** — after the commit is acknowledged, the changes survive crash, power loss and restart.
- **Write-ahead log (WAL / transaction log)** — a sequential record of changes written to stable storage *before* data pages are.
- **Commit acknowledgment** — control returning to the client; the moment durability is promised.
- **Checkpoint** — flushing dirty pages to the data file so recovery has less log to replay.
- **Redo / undo** — replay committed work missing from data pages; reverse uncommitted work found on them.
- **Eventual consistency** — replicas converge over time; reads may be stale.
- **Serialization failure** — the engine aborts a transaction because no serial order could explain the result; the app must retry.

### 3. How it works, step by step
How a commit becomes durable (the WAL protocol used by SQL Server and most engines):

1. The transaction modifies data pages **in memory** (buffer pool); the pages become "dirty".
2. For each change the engine appends a **log record** (before/after information, or enough to redo and undo) to the log buffer.
3. On `COMMIT`, the engine writes a commit record and **forces the log buffer to disk** (flush). Only after the flush completes does the engine return success to the client.
4. Dirty data pages are **not** written at commit; they are written later by checkpoints or the lazy writer.
5. If the machine crashes now, the log has everything needed; the data file may be missing changes, which is fine.
6. **Recovery after a crash:** the engine reads the log from the last checkpoint:
   - **Analysis/redo:** reapply logged changes whose effects are not on the data pages (this restores committed work and also repeats the work of uncommitted transactions).
   - **Undo:** roll back every transaction that has no commit record.
7. The database is now at a consistent state containing exactly the committed transactions.

Why this order: a sequential log append is cheap, a random page write is expensive; making only the log force synchronous gives durability at low cost.

Consistency in practice: constraints (PK, FK, CHECK, UNIQUE) are checked statement by statement; if one fails, the statement (or the whole transaction, depending on settings such as `XACT_ABORT`) is rolled back. Business invariants that constraints cannot express (sum of debits equals credits) are your responsibility inside the transaction.

### 4. Worked example
```sql
-- Durability and atomicity of a transfer
SET XACT_ABORT ON;           -- any runtime error aborts the whole transaction
BEGIN TRAN;
    UPDATE Account SET Balance = Balance - 100 WHERE AccountID = 1;
    UPDATE Account SET Balance = Balance + 100 WHERE AccountID = 2;
    INSERT INTO Ledger(FromAcct, ToAcct, Amount) VALUES (1, 2, 100);
COMMIT;                      -- returns only after the commit record is on disk
```

Crash timeline:
- Power fails **before** `COMMIT` returns: after restart the transfer is absent (undo). Neither balance changed.
- Power fails **after** `COMMIT` returned but before the data pages were flushed: after restart redo reapplies the three changes from the log. The client had been told "committed", and it is.

Observing the log: `SELECT log_reuse_wait_desc FROM sys.databases WHERE name = DB_NAME();` shows why the log cannot truncate; `DBCC SQLPERF(LOGSPACE)` shows its fill.

### 5. Under the hood
- SQL Server uses ARIES-style recovery: **analysis → redo → undo**, driven by log sequence numbers (LSNs). Each data page stores the LSN of the last change applied, so redo can skip changes already present.
- Traditional recovery undoes uncommitted work after redo; with **Accelerated Database Recovery (ADR)** in SQL Server 2019+, a persisted version store and a "sLog" make recovery and rollback near-instant regardless of transaction length.
- Durability can be loosened deliberately: **delayed durability** (`DELAYED_DURABILITY = ON`) acknowledges commit before the log flush, trading a small window of possible loss for throughput. In-memory tables with `SCHEMA_ONLY` durability lose data on restart by design.
- In distributed systems, atomicity across nodes needs a commit protocol (two-phase commit) and isolation is stated in terms of serializable *histories*; the CAP theorem forces a choice during partitions between refusing writes (consistency) and accepting divergent ones (availability).

### 6. Performance and design implications
- Commit latency is the log flush latency; put the log on fast, low-latency storage and keep it separate from data files.
- Group commit batches many transactions' flushes into one I/O; many tiny single-row autocommits still cost one flush each, so batch work into explicit transactions where semantics allow.
- Very large transactions lengthen recovery (without ADR) and hold locks and log space; chunk big modifications.
- Checkpoint frequency (recovery interval) trades steady-state I/O against crash recovery time.
- Some databases write new files per commit and need periodic cleanup; know whether many small commits cause bloat in your engine.
- Relaxing ACID for scale is legitimate but moves complexity into the application (compensations, idempotency).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `BEGIN TRAN / COMMIT / ROLLBACK` | Explicit transaction control | Any multi-statement change |
| `SET XACT_ABORT ON` | Abort whole transaction on runtime error | Standard in procedures |
| `ALTER DATABASE ... SET DELAYED_DURABILITY` | Async log flush | Throughput over strict durability |
| `CHECKPOINT` | Force dirty pages to disk | Controlled recovery time, tests |
| `ALTER DATABASE ... SET ACCELERATED_DATABASE_RECOVERY = ON` | Fast recovery and rollback | Long transactions, 2019+ |
| `sys.dm_tran_active_transactions` | See open transactions | Diagnosing long transactions |
| `DBCC SQLPERF(LOGSPACE)` | Log usage | Capacity checks |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Full durability (default) | You can tolerate losing the last moments (telemetry) → delayed durability |
| Strong consistency / ACID | Global scale across regions with latency limits → consider relaxed models |
| Saga with compensations | You only need one database; use a real transaction |
| Serializable isolation for invariants | Contention is high; prefer targeted locks or constraints |
| Two-phase commit | Participants are unreliable or slow; blocking coordinators hurt |

### 9. Common mistakes
- Assuming `COMMIT` means "on the data file" → misunderstanding recovery → durability is about the **log** being on stable storage.
- Believing constraints enforce all business consistency → silent invariant violations → enforce multi-row rules in the transaction (or with triggers/constraints you design).
- Using autocommit for a multi-step change → partial updates after a failure → wrap in one explicit transaction.
- Disabling the write cache safeguards or using volatile disk caches → "durable" data lost on power failure → use storage with battery-backed or write-through guarantees.
- Treating eventual consistency as ACID → stale reads break logic → design for it (version checks, idempotent writes).
- Ignoring serialization failures → user sees errors → retry the whole transaction.

### 10. 🎯 Interview kit
**30-second answer:** Durability means once commit returns, the data survives a crash. The engine achieves it with a write-ahead log: log records are flushed to disk before commit is acknowledged, data pages are written lazily, and recovery redoes committed work and undoes uncommitted work from the log.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.
Define ACID, walk the WAL commit path and crash recovery, mention delayed durability and ADR, and finish with distributed consistency (CAP, sagas).

**Likely follow-ups:**
- *Why log before data (WAL)?* → sequential log write is cheap and sufficient to rebuild pages.
- *Commit vs checkpoint?* → commit makes the transaction durable via the log; checkpoint flushes pages to shorten recovery.
- *What do redo and undo do?* → redo reapplies lost committed changes, undo reverses uncommitted ones.
- *ACID C vs CAP C?* → rule validity vs latest-write visibility.
- *Last seat booked by thousands?* → lock or conditional update (`UPDATE ... WHERE seats > 0`), check rows affected.
- *Saga?* → sequence of local transactions with compensating actions replacing distributed atomicity.

**Signals interviewers look for:** you tie durability to the log flush, know recovery phases, and can distinguish the two "consistencies".

### 11. Check yourself
- [ ] Draw the timeline of a commit with the log flush and a crash on either side.
- [ ] Explain redo versus undo in recovery.
- [ ] State what delayed durability trades away.
- [ ] Distinguish ACID consistency from CAP consistency.
- [ ] Say why dirty pages need not be written at commit.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is a database transaction and why is it useful?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00423_What_is_a_database_transaction_and_why_is_it_useful.md>)
- [Basic] [Why is durability often described as surviving a power failure or crash?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00500_Why_is_durability_often_described_as_surviving_a_power_failu.md>)
- [Intermediate] [Why might a large batch update be split into smaller transactions?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00482_Why_might_a_large_batch_update_be_split_into_smaller_transac.md>)
- [Deep] [What is Write-Ahead Logging (WAL) and why is it important for ACID?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00445_What_is_Write-Ahead_Logging_WAL_and_why_is_it_important_for_.md>)
- [Deep] [What is a serialization failure and how should an application respond to it?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00486_What_is_a_serialization_failure_and_how_should_an_applicatio.md>)
- [Deep] [How would you design a transactional workflow that must survive a mid-flight application deploy or restart?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00347_How_would_you_design_a_transactional_workflow_that_must_surv.md>)
- [Deep] [What consistency guarantees do you lose when you shard by customer and then need a cross-customer transaction?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00387_What_consistency_guarantees_do_you_lose_when_you_shard_by_cu.md>)
- [Deep] [How do you keep a cache and the database consistent when a transaction commits, avoiding stale reads?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00411_How_do_you_keep_a_cache_and_the_database_consistent_when_a_t.md>)

## Related concepts

- [Distributed Transactions and 2PC](<../../Database_Internals/DBMS_Concepts/Distributed_Transactions_and_2PC.md>) *(DBMS Concepts)*
- [LSM Write Path WAL and Compaction](<../../Database_Internals/Storage_Engines/Tiered_Storage_in_LSM_Engines/LSM_Write_Path_WAL_and_Compaction.md>) *(Storage Engines)*
- [Transaction Control and Prepared Transactions](<../../SQL_Language/SQL_Command_Types/Transaction_Control_and_Prepared_Transactions.md>) *(SQL Command Types)*
- [Row Locks Two-phase Locking and Lock Footprint](<../Locking/Row_Locks_Two-phase_Locking_and_Lock_Footprint.md>) *(Locking)*

## References

- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Generation in Source Systems › ACID
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Eventual Versus Strong Consistency
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › ACID Principles
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Durability
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Key Takeaways
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 16: Best Practices for T-SQL Development and Database Design › Fundamentals of Transactions
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Transactions and concurrency
- *Database Internals* — Alex Petrov — Transaction Processing and Recovery › Chapter 5. Transaction Processing and Recovery
- *Database Internals* — Alex Petrov — Transaction Processing and Recovery › Caching Semantics
- *Database Internals* — Alex Petrov — Distributed Transactions › Chapter 13. Distributed Transactions
- *Database Internals* — Alex Petrov — Distributed Transactions › Making Operations Appear Atomic
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 7. Transactions › The Meaning of ACID
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 7. Transactions › Atomicity
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 7. Transactions › Durability
- *Designing Data-Intensive Applications* — Martin Kleppmann — Single-Object and Multi-Object Operations
- *Azure Data Fundamentals* — Michael John Pena — Introduction to Data Representation › Characteristics of Structured Data
- *Azure Data Fundamentals* — Michael John Pena — Understanding Data Workloads › Strong consistency and ACID properties
- *SQL Server Tutorial* (YouTube series) — Part 58: Transactions in sql server and ACID Tests ([watch](https://www.youtube.com/watch?v=VLc4ewu6lUI))

