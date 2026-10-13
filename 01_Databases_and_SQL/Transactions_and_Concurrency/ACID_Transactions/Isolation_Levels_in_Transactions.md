---
id: "01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Isolation_Levels_in_Transactions"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions"
tools: ["MySQL"]
---

# Isolation Levels in Transactions

**L4 Advanced** · [↑ ACID Transactions](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** An isolation level decides which concurrency anomalies (dirty, non-repeatable, phantom reads and their relatives) a transaction is allowed to see, and stronger is not better: it is a price paid in blocking or retries.

### 1. The big picture
When many transactions run at once, each would like to believe it runs alone. Perfect isolation (serializable) delivers that illusion but costs concurrency. The SQL standard therefore defines four levels, each permitting fewer anomalies: READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, SERIALIZABLE. Real engines add snapshot-based variants, and their names do not always match behavior (PostgreSQL's REPEATABLE READ is actually snapshot isolation).

The practical rule from SQL Server practice: the default, READ COMMITTED, is right for the vast majority of workloads. Moving everything to REPEATABLE READ or SERIALIZABLE "because the data is important" is a mistake that creates blocking and deadlocks. Raise isolation for the specific transaction that needs it (actuarial calculations, invariants across rows), or use row versioning so readers do not block writers.

### 2. Core concepts and vocabulary
- **Dirty read** — reading data another transaction has changed but not committed; it may be rolled back.
- **Non-repeatable read** — reading the same row twice in one transaction and getting different values because another transaction committed a change between.
- **Phantom read** — re-running a range query and getting a different set of rows (rows inserted or deleted by others).
- **Lost update** — two read-modify-write cycles; one overwrites the other.
- **Write skew** — two transactions read overlapping data, write different rows, and together violate an invariant (two doctors both go off call).
- **Shared (S) lock** — taken by readers; compatible with other S locks.
- **Exclusive (X) lock** — taken by writers; incompatible with everything.
- **Row versioning** — keeping older row versions (in `tempdb` in SQL Server) so readers see a consistent snapshot without locks.
- **Snapshot isolation (SI)** — each transaction sees the data as of its start.
- **RCSI** — READ COMMITTED SNAPSHOT: statement-level versioned reads.
- **Serialization failure** — engine aborts a transaction that cannot be serialized; the app retries.

### 3. How it works, step by step
The anomaly matrix (standard definitions):

| Level | Dirty read | Non-repeatable read | Phantom |
|---|---|---|---|
| READ UNCOMMITTED | possible | possible | possible |
| READ COMMITTED | prevented | possible | possible |
| REPEATABLE READ | prevented | prevented | possible |
| SERIALIZABLE | prevented | prevented | prevented |
| SNAPSHOT (SQL Server) | prevented | prevented | prevented (for reads) but write skew possible |

How SQL Server implements them with locks:

1. **READ UNCOMMITTED**: readers take no shared locks (same as `NOLOCK`); can read uncommitted, even half-moved pages.
2. **READ COMMITTED (default)**: a reader takes an S lock on each row/page, reads it and **releases immediately**. Writers' X locks block it, so no dirty reads, but nothing stops changes after the read.
3. **REPEATABLE READ**: S locks on rows read are **held until the end of the transaction**, so those rows cannot change, but new rows in the range can appear (phantoms).
4. **SERIALIZABLE**: adds **key-range locks** so no row can be inserted into the range read.
5. **SNAPSHOT / RCSI**: readers use the version store; no S locks; writers still take X locks. SI is transaction-level (needs `ALLOW_SNAPSHOT_ISOLATION` and `SET TRANSACTION ISOLATION LEVEL SNAPSHOT`); RCSI replaces the default READ COMMITTED behavior statement by statement.

The choice is per transaction (`SET TRANSACTION ISOLATION LEVEL ...`) or per table hint, and database options change what READ COMMITTED means.

### 4. Worked example
Two query windows demonstrate a non-repeatable read and a phantom under READ COMMITTED.

```sql
-- Session 1                                   -- Session 2
BEGIN TRAN;
SELECT Balance FROM Account
 WHERE AccountID = 1;      -- 500
                                               UPDATE Account SET Balance = 400
                                                WHERE AccountID = 1;   -- autocommits
SELECT Balance FROM Account
 WHERE AccountID = 1;      -- 400: non-repeatable
COMMIT;
```

```sql
-- Phantom
BEGIN TRAN;
SELECT COUNT(*) FROM Orders WHERE CustomerID = 7;   -- 3
                                                    INSERT Orders(CustomerID) VALUES (7);
SELECT COUNT(*) FROM Orders WHERE CustomerID = 7;   -- 4 under RC/RR
COMMIT;
```
Repeat session 1 with `SET TRANSACTION ISOLATION LEVEL REPEATABLE READ`: session 2's `UPDATE` now **blocks** until session 1 ends, so the second read is 500; but the `INSERT` still succeeds (phantom). Under `SERIALIZABLE`, the `INSERT` blocks too.

Under `SNAPSHOT`, neither session blocks: session 1 keeps seeing 500 and 3 until it ends. If session 1 then tries to update the changed row it fails with an update-conflict error and must retry.

### 5. Under the hood
- Lock compatibility: S–S compatible, X incompatible with S and X; intent locks (IS, IX) at table/page level let the engine check coarse-granularity conflicts cheaply. Lock escalation converts many row locks to a table lock (around 5,000 locks per statement) to save memory, which can cause surprising blocking.
- Key-range locks in SERIALIZABLE lock the gaps between index keys, so they depend on an index covering the predicate; without an index the range is the whole table.
- Row versions live in the **version store** in `tempdb` (or in the user database with ADR's persisted version store). Each modified row carries a 14-byte version pointer; long-running snapshot transactions prevent cleanup and grow `tempdb`.
- PostgreSQL's engine is MVCC-only: READ COMMITTED takes a new snapshot per statement, REPEATABLE READ takes one per transaction (so it is snapshot isolation and still allows write skew), SERIALIZABLE uses SSI and may raise "could not serialize access", which the application must retry. Its READ UNCOMMITTED behaves like READ COMMITTED.
- MySQL InnoDB defaults to REPEATABLE READ with gap locks; Oracle defaults to READ COMMITTED with MVCC.

### 6. Performance and design implications
- Stronger locking isolation means readers block writers and writers block readers, raising wait times and deadlock rates.
- Versioning removes reader/writer blocking at the cost of `tempdb` I/O and space, and extra row overhead.
- `READ_COMMITTED_SNAPSHOT ON` is often the highest-leverage single setting for OLTP blocking; test because it changes semantics (readers no longer wait for in-flight writers).
- Measure with `sys.dm_tran_locks`, `sys.dm_os_wait_stats` (LCK_M_* waits), `sys.dm_tran_version_store_space_usage`.
- Reports that can tolerate approximation may use snapshot rather than `NOLOCK`, which risks missed or duplicated rows during page splits.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SET TRANSACTION ISOLATION LEVEL ...` | Sets level for the session | Specific transactions needing more |
| `ALTER DATABASE ... SET READ_COMMITTED_SNAPSHOT ON` | Versioned READ COMMITTED | Reduce reader/writer blocking |
| `ALTER DATABASE ... SET ALLOW_SNAPSHOT_ISOLATION ON` | Enables SNAPSHOT | Consistent reads over a transaction |
| `WITH (NOLOCK)` / `READUNCOMMITTED` | Dirty reads for that table | Rough monitoring only |
| `WITH (UPDLOCK, HOLDLOCK)` | U lock + range lock | Check-then-insert patterns |
| `DBCC USEROPTIONS` | Show session isolation | Verify settings |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| READ COMMITTED (default) | You need repeatable reads within a transaction |
| RCSI | Application relies on readers waiting for writers; heavy `tempdb` pressure |
| SNAPSHOT | Write skew matters; long transactions bloat version store |
| REPEATABLE READ | Wide scans; deadlocks from S-to-X conversion |
| SERIALIZABLE | High concurrency; use targeted locks or constraints instead |
| READ UNCOMMITTED | Anything involving money or decisions |

### 9. Common mistakes
- Defaulting to SERIALIZABLE for safety → massive blocking and deadlocks → use the lowest level that prevents the anomaly you actually care about.
- Spamming `NOLOCK` → missed/duplicate rows, reads of rolled-back data → use RCSI.
- Assuming REPEATABLE READ prevents phantoms → inserts still appear → SERIALIZABLE or a constraint.
- Assuming a name means the same everywhere (PostgreSQL REPEATABLE READ is snapshot) → bugs when porting → read the engine's docs.
- Not retrying serialization/update-conflict errors → user-visible failures → retry the whole transaction.
- Enabling snapshot with long-lived transactions → version store growth → keep transactions short, monitor `tempdb`.

### 10. 🎯 Interview kit
**30-second answer:** Isolation levels control which anomalies a transaction can observe: dirty reads, non-repeatable reads, phantoms. READ COMMITTED stops dirty reads, REPEATABLE READ also holds read locks, SERIALIZABLE adds range locks, and snapshot levels use row versions so readers do not block. Use the default unless a specific transaction needs more.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.
Define anomalies, draw the matrix, explain locks vs versions, then the cost and the overuse mistake.

**Likely follow-ups:**
- *Four standard levels?* → UNCOMMITTED, COMMITTED, REPEATABLE, SERIALIZABLE.
- *Non-repeatable read vs phantom?* → changed row vs changed row set.
- *Default levels?* → SQL Server and PostgreSQL: READ COMMITTED; MySQL InnoDB: REPEATABLE READ.
- *Why rarely READ UNCOMMITTED?* → dirty reads, inconsistent decisions.
- *"Could not serialize access"?* → retry the whole transaction.
- *PostgreSQL REPEATABLE READ?* → snapshot isolation; write skew still possible.

**Signals interviewers look for:** you tie each level to a concrete anomaly and mechanism, and you do not reflexively choose the strongest.

### 11. Check yourself
- [ ] Produce a non-repeatable read and a phantom with two sessions.
- [ ] Fill in the anomaly matrix from memory.
- [ ] Explain the difference between RCSI and SNAPSHOT.
- [ ] Say why SERIALIZABLE needs an index to be efficient.
- [ ] Describe write skew and one fix.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../../../TOOLS/MySQL/_index.md>)

## Practice questions

- [Basic] [What does the Isolation property of a transaction ensure?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00457_What_does_the_Isolation_property_of_a_transaction_ensure.md>)
- [Intermediate] [What is a dirty read and which isolation level allows it?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00390_What_is_a_dirty_read_and_which_isolation_level_allows_it.md>)
- [Intermediate] [Which isolation level is the default in MySQL InnoDB?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00427_Which_isolation_level_is_the_default_in_MySQL_InnoDB.md>)
- [Intermediate] [How does the isolation property help prevent a shopping-cart double-purchase problem?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00489_How_does_the_isolation_property_help_prevent_a_shopping-cart.md>)
- [Intermediate] [How does READ UNCOMMITTED behave in InnoDB versus SQL Server, and would you ever use it for reporting?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00360_How_does_READ_UNCOMMITTED_behave_in_InnoDB_versus_SQL_Server.md>)
- [Deep] [What is the difference between READ COMMITTED and REPEATABLE READ isolation levels?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00424_What_is_the_difference_between_READ_COMMITTED_and_REPEATABLE.md>)
- [Deep] [What is the trade-off between a higher isolation level and performance?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00430_What_is_the_trade-off_between_a_higher_isolation_level_and_p.md>)
- [Deep] [Under READ COMMITTED you saw duplicate inserts slip past an application-level check-then-insert; how would you make that operation safe?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00316_Under_READ_COMMITTED_you_saw_duplicate_inserts_slip_past_an_.md>)

## Related concepts

- [Range Locks and Retries](<../Locking/Range_Locks_and_Retries.md>) *(Locking)*
- [Phantoms and Double Booking](<../Isolation_Levels/Phantoms_and_Double_Booking.md>) *(Isolation Levels)*
- [Setting Isolation Levels](<../Isolation_Levels/Setting_Isolation_Levels/_index.md>) *(Isolation Levels)*

## References

- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Optimization › 10.11 #64 Using unnecessarily strong isolation levels
- *Designing Data-Intensive Applications* — Martin Kleppmann — Snapshot Isolation and Repeatable Read
- *Designing Data-Intensive Applications* — Martin Kleppmann — Snapshot Isolation and Repeatable Read › Repeatable read and naming confusion
- *Data Engineering Design Patterns* — Bartosz Konieczny — Idempotency Design Patterns › Read Uncommitted Isolation Level
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Snapshot Isolation (SI)

