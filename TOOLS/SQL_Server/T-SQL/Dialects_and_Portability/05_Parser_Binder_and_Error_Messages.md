---
id: "TOOLS/SQL_Server/T-SQL/Dialects_and_Portability/05_Parser_Binder_and_Error_Messages"
level: 5
type: "concept"
topic: "TOOLS/SQL_Server/T-SQL/Dialects_and_Portability"
tools: ["SQL_Server"]
---

# 5 · Parser, Binder and Error Messages

**L5** · [↑ Series index](<_index.md>) · Prev: [Same Syntax, Different Semantics](<04_Same_Syntax_Different_Semantics.md>) · Next: [Transactions and Concurrency Across Engines](<06_Transactions_and_Concurrency_Across_Engines.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A query is not "accepted or rejected" once — it passes through parser, binder, optimizer and executor, and the error message you receive tells you which stage refused it, which is the fastest way to decide whether the problem is dialect, metadata, version, or data.

### 1. The pipeline a batch goes through

```
 client text ──► [ 1 PARSE ] ──► [ 2 BIND / ALGEBRIZE ] ──► [ 3 OPTIMIZE ] ──► [ 4 EXECUTE ]
 (batch)          grammar          names, types,             plan search,        run operators,
                  T-SQL only       permissions               cost-based          locks, I/O
                  Msg 102,156…     Msg 207,208,195…          (plan cache)        Msg 245,515,547,1205…
```

| Stage | Question it answers | Needs the data/metadata? | Same text can pass here but fail later? |
|---|---|---|---|
| 1. **Parse** | Is the text valid T-SQL grammar? | No — pure syntax | Yes |
| 2. **Bind** | Do the objects, columns, functions exist and types fit? | Yes — catalog lookup | Yes |
| 3. **Optimize** | What is the cheapest plan? | Statistics and indexes | Usually no failure; may produce warnings |
| 4. **Execute** | Does the data cooperate? | Actual rows | – |

**Why this matters for dialects:** *syntax from another dialect always dies at stage 1.* Everything at stage 2 and later is about *your* database's objects and data, not the language. Reading the message number correctly saves hours.

### 2. Stage 1 — parser: grammar errors

| Message | Typical cause | Notes |
|---|---|---|
| **Msg 102** `Incorrect syntax near 'X'` | Wrong keyword / punctuation for T-SQL | The token named is where parsing *gave up*, often one token *after* the real mistake |
| **Msg 156** `Incorrect syntax near the keyword 'X'` | A reserved word in a wrong place | e.g., `SELECT … FROM t WHERE LIMIT 5` |
| **Msg 319** `Incorrect syntax near the keyword 'with'. … previous statement must be terminated with a semicolon` | CTE / `MERGE` / `THROW` not preceded by `;` | Habit: always terminate statements |
| **Msg 111** `'CREATE VIEW' must be the first statement in a query batch` | `CREATE VIEW/PROC/FUNCTION/TRIGGER` after other statements | `GO` separates; in ADF use separate activities |
| **Msg 1033** `ORDER BY invalid in views, inline functions, derived tables, subqueries, CTEs, unless TOP/OFFSET/FOR XML` | Wrapped query | Occurs when a tool wraps your text |

**Forensics of the "LIMIT" error.** `SELECT * FROM t LIMIT 5` produces `Incorrect syntax near '5'` — not near `LIMIT` — because after `FROM t` the parser accepts an optional alias, so `LIMIT` is consumed as an alias name and the *next* token `5` is the first thing that cannot follow. Rule: **start debugging at the token before the one named.**

**Other classic stage-1 failures when pasting another dialect**

| Pasted text | Message | Fix |
|---|---|---|
| ``SELECT `col` FROM t`` | `Incorrect syntax near 'col'` / invalid | `[col]` or `"col"` |
| `x::int` | `Incorrect syntax near ':'` | `CAST(x AS int)` |
| `INSERT … ON CONFLICT` / `ON DUPLICATE KEY` | `Incorrect syntax near 'ON'/'CONFLICT'` | `MERGE` |
| `col ILIKE 'a%'` | `Incorrect syntax near 'ILIKE'` | `LIKE` (collation) |
| `a \|\| b` (older versions) | `Incorrect syntax near '\|'` | `CONCAT`/`+` |
| `SELECT … FROM DUAL` | `Invalid object name 'DUAL'` | omit `FROM` |
| `CREATE TABLE IF NOT EXISTS …` | `Incorrect syntax near 'IF'` | `IF OBJECT_ID(N'dbo.t','U') IS NULL CREATE TABLE …` (or `DROP TABLE IF EXISTS` then create) |

### 3. Stage 2 — binder: names, types and permissions

| Message | Cause | Dialect angle |
|---|---|---|
| **Msg 207** `Invalid column name 'x'` | Column missing; alias used before its definition; wrong case under CS collation | T-SQL cannot reuse a `SELECT` alias in `WHERE`/`GROUP BY`/`HAVING` |
| **Msg 208** `Invalid object name 'x'` | Wrong schema/database; temp table out of scope; deferred creation | Other engines default schema differently (`public`, user schema) |
| **Msg 195** `'X' is not a recognized built-in function name` | Function from another dialect or newer version/compat level | `NVL`, `NOW`, `IFNULL`, `GENERATE_SERIES` on old compat level |
| **Msg 8120** `Column 'x' is invalid in the select list because it is not contained in an aggregate or GROUP BY` | Loose `GROUP BY` (MySQL permitted this before `ONLY_FULL_GROUP_BY`) | T-SQL strict |
| **Msg 209** `Ambiguous column name` | Same column from two joined tables | – |
| **Msg 468** `Cannot resolve the collation conflict …` | Comparing columns of different collations | Frequent when joining `tempdb` objects to user DB on a CS database |
| **Msg 229 / 262** `permission denied` | Missing `SELECT`/`EXECUTE` | Identity used by the *connector* |
| **Msg 2714** `There is already an object named 'x'` | Re-running DDL | No `CREATE TABLE IF NOT EXISTS`; use `CREATE OR ALTER` (programmable objects) or existence guard |

**Deferred name resolution** — a quirk specific to T-SQL batches: `CREATE PROCEDURE p AS SELECT * FROM NoSuchTable;` *succeeds* if the table doesn't exist yet (the object is resolved at first execution), but `SELECT * FROM NoSuchTable;` fails immediately. Column names for *existing* tables are still checked at create time. This is why a procedure may deploy successfully in ADF's pre-copy script and fail in the middle of a run.

### 4. Stage 3 — optimizer: plan warnings

No hard errors normally, but you see signals:

- **Plan warning** `Type conversion in expression … may affect "CardinalityEstimate" … CONVERT_IMPLICIT`
- **Missing index** hints in the plan XML
- **Plan reuse** depends on the *text hash and set options*: sessions with different `ANSI_NULLS`, `QUOTED_IDENTIFIER`, `DATEFORMAT`, `LANGUAGE`, `ARITHABORT` get separate plans. SSMS has `ARITHABORT ON` by default while many app drivers send `OFF` (historically), which is why "it's fast in SSMS and slow in the app" happens. See [Plan Cache and Reuse](<../../04_Query_Processing/Plan_Cache_and_Reuse/_index.md>) and [Parameter Sensitivity](<../../04_Query_Processing/Parameter_Sensitivity/_index.md>).
- **One statement at a time:** the optimizer plans each statement of a batch or procedure separately; local variables are **not sniffed** (it uses density averages), parameters are.

### 5. Stage 4 — executor: data and runtime errors

| Message | Cause | Remedy |
|---|---|---|
| **Msg 245** `Conversion failed when converting the varchar value 'x' to data type int` | Dirty data / implicit conversion | `TRY_CAST`, constraints, explicit casts |
| **Msg 8115** `Arithmetic overflow` | Value out of range of the type | wider type |
| **Msg 8134** `Divide by zero` | Division by zero | `NULLIF(d,0)` |
| **Msg 515** `Cannot insert NULL into column` | NOT NULL violation | defaults, null-handling |
| **Msg 547** FK / CHECK violation | Constraint | order loads parent-first |
| **Msg 2627 / 2601** duplicate key | UNIQUE / PK | dedupe, upsert |
| **Msg 8152** `String or binary data would be truncated` (2019+: **Msg 2628** with column name) | Value longer than column | wider column, trim, detect |
| **Msg 1205** deadlock victim | Cycle in locks | retry; see [Deadlocks](<../../05_Concurrency/Deadlocks/_index.md>) |
| **Msg 3960** snapshot update conflict | Concurrent update under SNAPSHOT | retry |
| **Msg 1222** lock timeout | `SET LOCK_TIMEOUT` | |
| **Msg 9002** transaction log full | Open transaction / growth | log management |

**Error severity** matters in tools: severity 11–16 are user-correctable errors (batch continues or aborts depending on type), 17–19 resource problems, ≥20 terminate the connection. A tool sees exceptions only for the batch.

### 6. Statement-abort vs batch-abort vs connection-abort

| Error class | What T-SQL does next (default `XACT_ABORT OFF`) |
|---|---|
| Most runtime errors (`Msg 8134`, constraint violation) | Statement aborted; **batch continues**; transaction stays open |
| Compile errors in a batch (syntax, missing column on existing table) | **Entire batch** not executed |
| Deadlock victim, severity ≥ 20 | Batch/transaction rolled back, may drop connection |

Consequence for pipelines: a Script/pre-copy activity with several statements may **partially** succeed unless wrapped in `TRY…CATCH` + `SET XACT_ABORT ON`.

```sql
SET XACT_ABORT ON;
BEGIN TRY
    BEGIN TRAN;
    DELETE dbo.Staging WHERE LoadDate < DATEADD(day,-7,GETDATE());
    INSERT dbo.Fact SELECT * FROM dbo.StagingNew;
    COMMIT;
END TRY
BEGIN CATCH
    IF @@TRANCOUNT > 0 ROLLBACK;
    THROW;           -- re-raise so the calling tool sees a failure
END CATCH;
```
Contrast: PostgreSQL aborts the *whole transaction* on any error (`current transaction is aborted, commands ignored until end of transaction block`).

### 7. Compile-time vs run-time detection — a decision table

| Symptom | First stage to suspect | First thing to try |
|---|---|---|
| Fails instantly, nothing runs | Parser | Compare with the grammar; strip to the failing clause |
| "Invalid column/object" | Binder | Check database context (`SELECT DB_NAME()`), schema, case/collation |
| Runs in SSMS, fails in tool | Client/wrapper | Look for `GO`, `USE`, multiple result sets, wrapped subquery |
| Runs on dev, fails on prod | Version/compat/platform | `SELECT @@VERSION`, `compatibility_level`, `SERVERPROPERTY('EngineEdition')` |
| Works for 1M rows then fails | Executor | `TRY_CAST` bad value discovery; find the row |
| Fast, then slow after deploy | Optimizer/plan cache | Plan comparison, Query Store |

### 8. Worked example — reading a failing ADF activity

Error returned in the activity output:

```
Failure happened on 'Source' side.
ErrorCode=SqlOperationFailed,
'Type=System.Data.SqlClient.SqlException,Message=Incorrect syntax near the keyword 'ORDER'.
Incorrect syntax near ')'. ...
```

Diagnosis path:
1. `SqlException` → the source is a SQL Server family engine ⇒ T-SQL parser rejected it (stage 1).
2. Message mentions `ORDER` → the query likely has `ORDER BY` inside a subquery/CTE.
3. Is the activity using a **partition option**? ADF appends predicates / wraps → derived table → `Msg 1033`-family.
4. Fix: remove `ORDER BY` from the query (the sink does not need order), or add `TOP 100 PERCENT` (ignored by the optimizer; avoid), or **move the logic into a view / stored procedure** and select from that.

### 9. Under the hood

- The parser works over *batches*. `GO` is not in the grammar — SSMS/`sqlcmd` split on it. The server only ever sees one batch per request.
- Binding resolves names via the metadata cache; it consults the catalog and checks permissions along an ownership chain. Cached plans are invalidated by schema changes (`sys.sql_modules`, statistics updates).
- Plan compilation is cost-based and can time out after exploring a fraction of the search space (`TimeOut` reason in plan XML). Complex T-SQL — many joins, large `OR` trees — may get a "good enough" plan.
- Error numbers are stable and documented in `sys.messages` (`SELECT * FROM sys.messages WHERE message_id = 102 AND language_id = 1033`).

### 10. Performance and design implications

- Validate *without executing*: `SET PARSEONLY ON` (syntax only), `SET NOEXEC ON` (compile, don't run) or `sys.dm_exec_describe_first_result_set(@sql, NULL, 0)`.
- Prefer **parameterised** queries over concatenated strings: avoids cache bloat and injection; in ADF, parameters belong in the *stored procedure activity* or *Script parameters*, not string-concatenated into `sqlReaderQuery`.
- Put non-trivial logic behind views/procs so that wrappers only see simple `SELECT`s.

### 11. Common mistakes

- Blaming "ADF" for what is a SQL Server parse error.
- Fixing the token named in `Msg 102` rather than the one *before* it.
- Re-running a failed multi-statement script without realising earlier statements committed.
- Testing only on SSMS — a different session context (set options, default database, collation of tempdb) from the production job.

### 12. 🎯 Interview kit

**30-second answer:** "Parse → bind → optimize → execute. Syntax from another dialect fails at parse with Msg 102/156; names and types at bind with 207/208/195; data problems at execution with 245/515/547. The message number tells me which layer to investigate."

**Likely follow-ups:** *Why did the procedure create but fail at run time?* → deferred name resolution. *Why is it fast in SSMS and slow from the app?* → different SET options ⇒ different plans/sniffing. *What's the difference between `PARSEONLY` and `NOEXEC`?* → syntax only vs compile without execute (binding errors visible).

### 13. Check yourself

- [ ] Map Msg 102, 207, 245, 1205 to a stage.
- [ ] Explain why `LIMIT 5` reports an error at `5`.
- [ ] State what happens to a three-statement batch if statement 2 raises Msg 8134 with `XACT_ABORT OFF` and ON.
- [ ] List three reasons a query works in SSMS but not in a pipeline.

**Where this connects:** [Execution Plans](<../../04_Query_Processing/Execution_Plans/_index.md>) · [Plan Cache and Reuse](<../../04_Query_Processing/Plan_Cache_and_Reuse/_index.md>) · [MERGE Pitfalls and Error Handling](<../MERGE_Pitfalls_and_Error_Handling.md>) · Next → [Transactions and Concurrency Across Engines](<06_Transactions_and_Concurrency_Across_Engines.md>)
<!-- explainer:end -->
