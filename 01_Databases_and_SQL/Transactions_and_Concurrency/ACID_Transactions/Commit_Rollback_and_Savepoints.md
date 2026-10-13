---
id: "01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Commit_Rollback_and_Savepoints"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions"
tools: []
---

# Commit Rollback and Savepoints

**L4 Advanced** · [↑ ACID Transactions](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** `BEGIN TRAN` opens a unit of work, `COMMIT` makes it permanent, `ROLLBACK` undoes it, and savepoints let you undo only part of it, but SQL Server's nesting and error rules differ from what most people assume.

### 1. The big picture
Transaction control language (TCL) is the small set of statements that delimit units of work: begin, commit, rollback and, in some engines, savepoints. Every statement runs inside a transaction; the question is who sets the boundaries. In SQL Server the default is **autocommit** (each statement is its own transaction). Explicit transactions group statements; implicit transactions (`SET IMPLICIT_TRANSACTIONS ON`) start one automatically after the previous ends, which is how Oracle behaves and a frequent source of "why is my session blocking?" surprises.

Knowing the commands is easy. The subtle parts are what happens with errors (does the transaction survive?), nested `BEGIN TRAN`, which statements cannot be rolled back, and how batches relate to transactions.

### 2. Core concepts and vocabulary
- **Autocommit** — each statement commits itself.
- **Explicit transaction** — delimited by `BEGIN TRAN` and `COMMIT`/`ROLLBACK`.
- **Implicit transaction** — a transaction starts automatically; you must commit.
- **`@@TRANCOUNT`** — number of open `BEGIN TRAN` levels for the session.
- **Savepoint** — a named point inside a transaction to roll back to (`SAVE TRAN name`).
- **Batch** — statements sent and parsed together (`GO` is a client tool delimiter, not T-SQL); a batch is not a transaction.
- **`XACT_ABORT`** — setting that makes a runtime error abort and roll back the whole transaction.
- **Doomed transaction** — one that can only be rolled back (`XACT_STATE() = -1`).
- **DDL vs DML** — structure changes vs data changes; in SQL Server both are transactional, in some engines DDL auto-commits.
- **Checkpoint** — writes dirty pages to disk; not the same as commit.

### 3. How it works, step by step
1. `BEGIN TRAN` increments `@@TRANCOUNT` to 1 and starts logging with undo information.
2. Each DML statement takes locks and writes log records; nothing is visible to others at READ COMMITTED until commit (if locking-based).
3. `COMMIT` decrements `@@TRANCOUNT`. Only when it reaches **0** does the engine write the commit record, flush the log and release locks. An inner `COMMIT` of a nested `BEGIN TRAN` commits nothing.
4. `ROLLBACK` (without a name) rolls back **everything** to the outermost `BEGIN TRAN` and sets `@@TRANCOUNT` to 0, regardless of nesting.
5. `SAVE TRAN sp1` marks a point; `ROLLBACK TRAN sp1` undoes work after it, leaves earlier work, and keeps the transaction open and locks held. You must still commit or roll back fully afterward.
6. Errors: a statement-level error (constraint violation) rolls back only that statement by default and the transaction continues, unless `XACT_ABORT ON`, in which case it rolls back the whole transaction. A batch-level error (some conversion/compile errors) ends the batch and can leave a transaction open.
7. A trigger that raises an error (or issues `ROLLBACK`) causes the triggering statement's transaction to be rolled back; the trigger runs inside the caller's transaction.
8. Closing the connection with an open transaction rolls it back.

### 4. Worked example
```sql
SET XACT_ABORT ON;
BEGIN TRAN;
    INSERT dbo.Orders(OrderID, CustomerID) VALUES (1001, 7);

    SAVE TRAN after_header;                 -- savepoint

    INSERT dbo.OrderLines(OrderID, LineNo, ProductID, Qty)
    VALUES (1001, 1, 55, 3);

    -- business rule: line not allowed, undo only the lines
    IF NOT EXISTS (SELECT 1 FROM dbo.Stock WHERE ProductID = 55 AND OnHand >= 3)
        ROLLBACK TRAN after_header;         -- header insert survives

    SELECT @@TRANCOUNT AS open_levels;      -- still 1
COMMIT;                                     -- header committed, lines not
```

Nesting trap:
```sql
BEGIN TRAN;            -- trancount 1
  BEGIN TRAN;          -- trancount 2
  COMMIT;              -- trancount 1, nothing committed yet
ROLLBACK;              -- trancount 0, everything undone
```
A stored procedure that does its own `BEGIN TRAN ... ROLLBACK` when called inside the caller's transaction rolls back the caller's work too. The safe pattern: check `@@TRANCOUNT`; if already inside a transaction, use a savepoint instead of a nested begin.

### 5. Under the hood
*(optional)* Savepoints work because the log is a chain of records per transaction; rolling back to a savepoint walks the chain back to the marker and writes compensation records. Locks acquired after the savepoint are **not** released at rollback-to-savepoint in SQL Server (they are held until the transaction ends), which can surprise people expecting reduced blocking.

A transaction in SQL Server is tied to the session, not the batch: a transaction may span multiple batches, and a batch may contain several transactions. If a batch fails to *parse*, none of it runs; if it fails at run time, earlier statements already executed stay in effect unless wrapped in a transaction with `XACT_ABORT`.

Commit versus checkpoint: commit forces the **log**; checkpoint writes **data pages** to shorten crash recovery. Committed data may exist only in the log until a checkpoint, which is why the log must be safe.

### 6. Performance and design implications
- Open transactions hold locks and block log truncation; the single most common cause of a growing log is an old open transaction (`DBCC OPENTRAN`).
- Many tiny autocommit statements each pay a log flush; batch them in a transaction when atomicity allows.
- Huge transactions are costly to roll back (rollback is often as slow as the work itself) and bloat the log; chunk them.
- Avoid user think-time and external calls inside transactions.
- Implicit transactions plus a forgotten commit create long blockers.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `BEGIN TRAN` / `BEGIN TRANSACTION` | Start explicit transaction | Multi-statement unit |
| `COMMIT [TRAN]` | Commit when outermost | End successful work |
| `ROLLBACK [TRAN [name]]` | Undo all / to savepoint | Error paths |
| `SAVE TRAN name` | Create savepoint | Partial undo |
| `@@TRANCOUNT`, `XACT_STATE()` | Inspect nesting/state | CATCH blocks |
| `SET XACT_ABORT ON` | Whole-transaction abort on error | Default in procedures |
| `SET IMPLICIT_TRANSACTIONS ON` | Auto-begin transactions | Compatibility only |
| `DBCC OPENTRAN` | Oldest open transaction | Log growth, blocking |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Explicit transaction | Single-statement change (already atomic) |
| Savepoint | You can restructure into separate transactions; locks stay held anyway |
| `XACT_ABORT ON` | You deliberately want partial progress after an error |
| `TRY...CATCH` with `XACT_STATE()` check | Nested procedures that might all roll back |
| Autocommit for bulk loads in batches | You need all-or-nothing |
| Implicit transactions | New code; prefer explicit |

### 9. Common mistakes
- Assuming an inner `COMMIT` commits → inner "commit" does nothing, outer `ROLLBACK` undoes it → commit only at the outermost level.
- Bare `ROLLBACK` inside a nested procedure → whole caller transaction lost, `@@TRANCOUNT` mismatch error → use savepoints or `TRANCOUNT` checks.
- Full `ROLLBACK` when only part should be undone → lost valid work → `ROLLBACK TO savepoint`.
- No `XACT_ABORT` → error leaves transaction open and locks held → set it and roll back in CATCH.
- Assuming DDL cannot be rolled back (true in some engines) → in SQL Server DDL is transactional; know your engine.
- Confusing `GO` batches with transactions → expecting rollback of earlier batches → wrap in explicit transaction.

### 10. 🎯 Interview kit
**30-second answer:** TCL is `BEGIN`, `COMMIT`, `ROLLBACK` and savepoints. Commit makes work durable by flushing the log and releasing locks; rollback undoes using the log; a savepoint rolls back only the part after it while the transaction stays open. In SQL Server nested `BEGIN TRAN` only counts levels, and a bare rollback undoes everything.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.
Show autocommit vs explicit, the trancount rules, error behavior with `XACT_ABORT`, then the stored-procedure rollback pattern.

**Likely follow-ups:**
- *What happens to changes on rollback?* → undone via log, locks released.
- *Savepoint use?* → partial rollback inside a long operation.
- *DDL vs DML transactions?* → engine-specific; SQL Server DDL is transactional, Oracle commits implicitly.
- *Trigger error?* → the triggering statement and transaction roll back.
- *Commit vs checkpoint?* → log force vs data page flush.
- *What does SET TRANSACTION do (standard SQL)?* → sets isolation and access mode for the next transaction; SQL Server uses `SET TRANSACTION ISOLATION LEVEL`.
- *Redo/undo after a crash?* → redo committed, undo uncommitted from the log.

**Signals interviewers look for:** you know nested-transaction semantics and error handling, not just the keywords.

### 11. Check yourself
- [ ] Predict `@@TRANCOUNT` after a nested begin/commit/rollback sequence.
- [ ] Use a savepoint to undo part of a transaction.
- [ ] Explain XACT_ABORT's effect on a constraint error.
- [ ] Say why a procedure should not unconditionally roll back.
- [ ] Find and explain the oldest open transaction.
<!-- explainer:end -->

## Practice questions

- [Basic] [What happens to the changes made by a transaction if it is rolled back?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00324_What_happens_to_the_changes_made_by_a_transaction_if_it_is_r.md>)
- [Basic] [What SQL commands make up Transaction Control Language (TCL)?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00466_What_SQL_commands_make_up_Transaction_Control_Language_TCL.md>)
- [Intermediate] [What is a SAVEPOINT in a transaction and when would you use one?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00357_What_is_a_SAVEPOINT_in_a_transaction_and_when_would_you_use_.md>)
- [Intermediate] [Can you roll back a TRUNCATE TABLE statement and why or why not?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00459_Can_you_roll_back_a_TRUNCATE_TABLE_statement_and_why_or_why_.md>)
- [Intermediate] [What is the difference between an implicit and an explicit transaction?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00478_What_is_the_difference_between_an_implicit_and_an_explicit_t.md>)
- [Intermediate] [How would you audit that every code path either commits or rolls back its transaction, even on exceptions?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00404_How_would_you_audit_that_every_code_path_either_commits_or_r.md>)
- [Deep] [What is the difference between COMMIT and CHECKPOINT in a database?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00481_What_is_the_difference_between_COMMIT_and_CHECKPOINT_in_a_da.md>)
- [Deep] [Why might an autonomous transaction be exactly right for audit logging even when the main transaction rolls back?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00372_Why_might_an_autonomous_transaction_be_exactly_right_for_aud.md>)

## Related concepts

- [Transaction Control and Prepared Transactions](<../../SQL_Language/SQL_Command_Types/Transaction_Control_and_Prepared_Transactions.md>) *(SQL Command Types)*
- [Read Committed Locking Anomalies](<../../../TOOLS/SQL_Server/SQL_Server_Platform/Read_Committed_Locking_Anomalies.md>) *(SQL Server Platform)*
- [Distributed Transactions and 2PC](<../../Database_Internals/DBMS_Concepts/Distributed_Transactions_and_2PC.md>) *(DBMS Concepts)*

## References

- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Chapter 8. Queries, Modeling, and Transformation › Know how your database handles commits
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Transactions in Stored Procedures
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Explanation
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Transaction Control Commands
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › ROLLBACK TRANSACTION
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Programmable objects
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Datastore Field Guide › Transactions
- *Azure Data Fundamentals* — Michael John Pena — Relational Data Concepts › A Practical Example
- *SQL Server Tutorial* (YouTube series) — Part 57: Transactions in sql server ([watch](https://www.youtube.com/watch?v=shkt9Z5Gz-U))

