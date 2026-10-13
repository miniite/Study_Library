---
id: "TOOLS/SQL_Server/T-SQL/MERGE_Pitfalls_and_Error_Handling"
level: 4
type: "concept"
topic: "TOOLS/SQL_Server/T-SQL"
tools: ["SQL_Server"]
---

# MERGE Pitfalls and Error Handling

**L4 Advanced** · [↑ T-SQL](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** `MERGE` does insert/update/delete in one statement, but it has well-known concurrency and correctness traps — so use it deliberately, serialize it properly, and wrap data changes in transactions with `TRY…CATCH` and `XACT_ABORT`.

### 1. The big picture
Loading data usually needs "upsert" logic: update rows that exist, insert those that don't, optionally delete what vanished from the source. `MERGE` (SQL Server 2008+) expresses this in a single statement comparing a *target* with a *source*. It looks elegant, but it has a long list of historical bugs and a race condition under concurrency, so many teams ban it in OLTP code and keep it for controlled ETL.

This lesson sits at the junction of DML, transactions (atomicity, isolation) and error handling: whichever pattern you choose, a failed multi-statement change must leave the data consistent.

### 2. Core concepts and vocabulary
- **Target / source** — table being modified / rowset providing changes.
- **Match condition** — `ON` predicate pairing rows; must identify at most one source row per target row.
- **`WHEN MATCHED`**, **`WHEN NOT MATCHED [BY TARGET]`**, **`WHEN NOT MATCHED BY SOURCE`** — action clauses.
- **`OUTPUT` / `$action`** — returns affected rows and which action touched them.
- **Atomicity** — all-or-nothing; each statement is atomic, multi-statement units need a transaction.
- **`XACT_ABORT`** — makes run-time errors abort and roll back the whole transaction.
- **`TRY…CATCH`** — structured error handling; **`THROW`** re-raises.
- **Doomed transaction** — `XACT_STATE() = -1`: only rollback is allowed.
- **Race condition** — two sessions both see "not matched" and both insert.

### 3. How it works, step by step
1. Source and target are joined (full outer join logically) on the `ON` predicate.
2. Each joined row is classified: matched, only-in-source, only-in-target.
3. Applicable `WHEN` clauses fire (at most one action per row).
4. Changes are applied inside the statement's implicit transaction; triggers fire once per action type.
5. `OUTPUT` returns inserted/deleted images.

If the same target row matches **more than one** source row, SQL Server raises error 8672 ("MERGE statement attempted to UPDATE or DELETE the same row more than once"): deduplicate the source first.

### 4. Worked example
```sql
SET XACT_ABORT ON;
BEGIN TRY
    BEGIN TRAN;

    MERGE dbo.Customer WITH (HOLDLOCK) AS t           -- serialize key range
    USING (SELECT CustomerID, Name, Email FROM stg.Customer) AS s
          ON t.CustomerID = s.CustomerID
    WHEN MATCHED AND (t.Name <> s.Name OR t.Email <> s.Email) THEN
         UPDATE SET t.Name = s.Name, t.Email = s.Email
    WHEN NOT MATCHED BY TARGET THEN
         INSERT (CustomerID, Name, Email) VALUES (s.CustomerID, s.Name, s.Email)
    OUTPUT $action, inserted.CustomerID;               -- semicolon is mandatory

    COMMIT;
END TRY
BEGIN CATCH
    IF XACT_STATE() <> 0 ROLLBACK;
    THROW;                                             -- preserve original error
END CATCH;
```
What to notice: `HOLDLOCK` (= `SERIALIZABLE`) takes key-range locks so concurrent sessions cannot both decide "not matched"; the extra `AND` in `WHEN MATCHED` avoids pointless updates; `THROW` without arguments re-raises the original error number and message.

Alternative without `MERGE`:
```sql
UPDATE t SET ... FROM dbo.Customer t JOIN stg.Customer s ON ...;
INSERT dbo.Customer (...) SELECT ... FROM stg.Customer s
WHERE NOT EXISTS (SELECT 1 FROM dbo.Customer t WHERE t.CustomerID = s.CustomerID);
```
(wrapped in one transaction, with `UPDLOCK, HOLDLOCK` hints for concurrency).

### 5. Under the hood
- `MERGE` compiles into a single plan with a *Merge/Hash join* then a *Clustered Index Merge* operator that decides insert/update/delete per row. That combined operator is where several bugs lived (filtered indexes, indexed views, triggers with `OUTPUT`, temporal tables, etc.).
- **Locking:** it takes `U` locks while probing, then converts to `X`; without `HOLDLOCK` the gap isn't locked, so two sessions can both pass the "not matched" test and one fails with a PK violation (2627/2601) — or both succeed if the key isn't unique.
- Triggers see all affected rows via `inserted`/`deleted` once per action; `$action` in `OUTPUT` identifies which.
- Errors split into **statement-terminating** (transaction can continue unless `XACT_ABORT ON`), **batch-terminating** and **connection-terminating**. Compile-time errors (missing object in a deferred-name-resolution case) abort the batch before `CATCH` can run at the same level.
- `TRY…CATCH` does not catch severity ≥ 20 that kill the connection, or errors with severity < 11.

### 6. Performance and design implications
- Unique index on the match key is essential; otherwise each source row scans the target.
- Batch large loads (e.g. 10k–100k rows) to bound log growth and lock escalation (5,000-lock threshold).
- Prefer a staging table so the source is pre-validated and de-duplicated.
- `WHEN NOT MATCHED BY SOURCE THEN DELETE` against a *partial* source deletes legitimate rows — always filter the target side.
- Measure with `STATISTICS IO`, Extended Events (`sql_statement_completed`), and the `sys.dm_tran_locks` DMV.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `MERGE … WITH (HOLDLOCK)` | key-range serializable upsert | concurrent upserts |
| `SET XACT_ABORT ON` | rollback on any run-time error | all transactional procs |
| `BEGIN TRY/CATCH`, `THROW` | handle and re-raise | error handling |
| `XACT_STATE()` | 1 committable, -1 doomed, 0 none | in `CATCH` before commit/rollback |
| `ERROR_NUMBER()/MESSAGE()/LINE()/PROCEDURE()` | diagnostic info | logging |
| `RAISERROR` vs `THROW` | legacy vs modern raise | prefer `THROW` |
| `OUTPUT … INTO` | capture changes | audit/ETL |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Controlled batch ETL with de-duplicated staging | High-concurrency OLTP upsert without `HOLDLOCK` |
| You need `$action` audit via `OUTPUT` | Tables with triggers/indexed views/temporal on older builds |
| Single readable statement is valuable | Separate `UPDATE`+`INSERT` are simpler to reason about |
| `THROW` for re-raise | Swallowing errors in `CATCH` without logging |

### 9. Common mistakes
- Duplicate keys in source → error 8672 → `ROW_NUMBER()` dedupe.
- No `HOLDLOCK`/`UPDLOCK` → race, PK violations → add hint or retry.
- Missing semicolon before `MERGE`/`THROW` → syntax error → terminate previous statement.
- `CATCH` that does nothing → silent failure → log and re-raise.
- Committing in `CATCH` when `XACT_STATE() = -1` → error 3930 → rollback.
- Using `@@ERROR` after another statement → value reset → capture immediately or use `TRY…CATCH`.
- Delete clause without target filter → data loss.

### 10. 🎯 Interview kit
**30-second answer:** `MERGE` combines insert/update/delete by comparing source to target. It's convenient but has concurrency races and past bugs, so use `HOLDLOCK`, unique match keys and de-duplicated sources, inside a transaction with `XACT_ABORT ON` and `TRY…CATCH` + `THROW`.

**Structure a longer answer:** clarify → semantics of clauses → race condition & fix → alternatives → error handling layout.

**Likely follow-ups:** *Why error 8672?* → source duplicates. *Difference RAISERROR/THROW?* → THROW re-raises, severity 16, needs prior semicolon. *What if XACT_ABORT off?* → some errors leave transaction open. *Alternative to MERGE?* → UPDATE then INSERT with locking hints.

**Signals interviewers look for:** awareness of the race and bugs, transactional discipline, `XACT_STATE`.

### 11. Check yourself
- [ ] Reproduce a duplicate-key race with two sessions, then fix with `HOLDLOCK`.
- [ ] Trigger error 8672 and fix it.
- [ ] Write a `CATCH` block that rolls back safely and re-raises.
- [ ] Explain doomed transactions.
<!-- explainer:end -->

## Implementations (tools)

- [SQL Server](<../_index.md>)

## Practice questions

- [Intermediate] [What is TRY CATCH error handling in T-SQL and how do you use it?](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42912_What_is_TRY_CATCH_error_handling_in_T-SQL_and_how_do_you_use.md>)
- [Deep] [You are seeing update conflicts (error 3960) after enabling SNAPSHOT isolation. Explain why and how you would handle them in code.](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42775_You_are_seeing_update_conflicts_error_3960_after_enabling_SN.md>)
- [Deep] [Explain lock escalation in SQL Server, when it triggers, and how you would prevent it from escalating to a full table lock during a bulk update.](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42779_Explain_lock_escalation_in_SQL_Server_when_it_triggers_and_h.md>)
- [Deep] [How do you split a large batch DELETE or UPDATE to avoid lock escalation and keep the transaction log from exploding?](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42798_How_do_you_split_a_large_batch_DELETE_or_UPDATE_to_avoid_loc.md>)
- [Deep] [Explain optimistic concurrency using a rowversion column and how the application detects a conflicting update.](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42824_Explain_optimistic_concurrency_using_a_rowversion_column_and.md>)
- [Deep] [How does the transaction log work with checkpoints, and how do the FULL SIMPLE and BULK_LOGGED recovery models affect log growth?](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42828_How_does_the_transaction_log_work_with_checkpoints_and_how_d.md>)
- [Deep] [Walk me through investigating a report where the same procedure sometimes returns in 200ms and sometimes in 20 seconds.](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42861_Walk_me_through_investigating_a_report_where_the_same_proced.md>)
- [Expert] [What are the UPDLOCK HOLDLOCK and READPAST table hints and give a real scenario for each in a queue-processing pattern.](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42780_What_are_the_UPDLOCK_HOLDLOCK_and_READPAST_table_hints_and_g.md>)

## Related concepts

- [Read Committed Locking Anomalies](<../SQL_Server_Platform/Read_Committed_Locking_Anomalies.md>) *(SQL Server Platform)*
- [SKIP LOCKED Job Queues](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/SKIP_LOCKED_Job_Queues.md>) *(SQL Fundamentals)*
- [Large Batch Updates](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Large_Batch_Updates.md>) *(SQL Fundamentals)*

## References

- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Managing Transactions › Using the XACTABORT Setting
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Managing Transactions › Using TRY . . . CATCH with Transactions
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Managing Transactions › Thinking About Performance
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › Debugging and Best Practices
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › Debugging T-SQL Code
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Best Practices in Error Handling
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Error Handling with TRY...CATCH
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Transactions and concurrency
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 3 Methods For Capturing Query Performance Metrics
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 16 Blocking And Blocked Processes
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — SQL Trace
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Extended Events
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — CXPACKET
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — PAGEIOLATCH
- *Designing Data-Intensive Applications* — Martin Kleppmann — I

