---
id: "TOOLS/SQL_Server/06_T-SQL_Programming/Programmable_Objects"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Procedures Functions and Triggers

**L4 Advanced** · [↑ T-SQL Programming](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Programmable objects package T-SQL logic on the server: variables, batches and flow control are the building blocks; stored procedures, functions and triggers are the stored units; temp tables, dynamic SQL and TRY…CATCH error handling are the tools. Each one has a performance profile you need to know before you use it.

### 1. The big picture
Queries describe *data*. Programmable objects describe *process*: "validate these inputs, start a transaction, update three tables, log the result, and roll back cleanly on any error." T-SQL provides procedural features for this, and SQL Server stores the logic as database objects so applications call one name instead of sending many statements.

The detailed vendor-neutral lessons on stored procedures, triggers and views live in the Database Objects block. This lesson is the SQL Server T-SQL view: syntax, error-handling patterns, and the engine-specific behavior that makes a scalar UDF slow or a trigger dangerous.

### 2. Core concepts and vocabulary
- **Variable**: `DECLARE @x int = 5;` holds one value for the rest of the *batch*. Set with `SET` (one variable, errors if a subquery returns many rows) or `SELECT` (several at once; with many rows, silently keeps the last one).
- **Batch**: statements sent and compiled together. A compile error stops the whole batch; variables don't survive past `GO`.
- **Flow control**: `IF … ELSE`, `WHILE` (with `BREAK` / `CONTINUE`), `BEGIN … END` blocks, `RETURN`, `WAITFOR`.
- **Cursor**: an object that walks a result row by row. `LOCAL FAST_FORWARD` (read-only, forward-only) is the cheapest kind.
- **Temporary table** (`#t`): real table in tempdb, visible to the session (and to procedures it calls), has statistics and indexes. `##t` is global.
- **Table variable** (`@t`): table in tempdb scoped to the batch; no column statistics; not affected by `ROLLBACK`.
- **Dynamic SQL**: building a statement as a string and running it with `EXEC` or, better, `sp_executesql` with parameters.
- **SQL injection**: an attack where user input is concatenated into dynamic SQL and becomes code.
- **Stored procedure**: a stored batch with parameters (input and `OUTPUT`), a return code, and possibly result sets.
- **Scalar UDF**: function returning one value. **Inline TVF**: returns a table from one `SELECT` (inlined). **Multi-statement TVF**: fills and returns a table variable.
- **Trigger**: code that runs automatically on an event. **DML triggers**: `AFTER` (after the change, inside its transaction) or `INSTEAD OF` (replaces the change). **DDL triggers** and **logon triggers** also exist.
- **inserted / deleted**: pseudo-tables available in triggers (and `OUTPUT`) with the new and old row images.
- **TRY…CATCH**: structured error handling. **THROW** raises or re-raises an error; **RAISERROR** is the older form.
- **XACT_STATE()**: 1 = active committable transaction, −1 = doomed (must roll back), 0 = none.

### 3. How it works, step by step
**Error handling pattern for a procedure that modifies data:**
1. `SET NOCOUNT ON; SET XACT_ABORT ON;` — XACT_ABORT makes most runtime errors abort and roll back the transaction instead of leaving it half-done.
2. `BEGIN TRY` → `BEGIN TRAN` → statements → `COMMIT` → `END TRY`.
3. `BEGIN CATCH` → if `@@TRANCOUNT > 0` roll back → log details with the error functions → `THROW;` to re-raise to the caller → `END CATCH`.

Functions available only inside CATCH: `ERROR_NUMBER()`, `ERROR_MESSAGE()`, `ERROR_SEVERITY()`, `ERROR_STATE()`, `ERROR_LINE()`, `ERROR_PROCEDURE()`.

What TRY…CATCH does *not* catch: compile errors in the same batch (syntax, and missing-object errors at the same level); errors with severity ≤ 10 (warnings); severity ≥ 20 that ends the connection; and attention events such as a client timeout (another reason for XACT_ABORT ON: a client timeout otherwise leaves an open transaction).

**Trigger execution:**
1. The DML statement modifies rows (for `AFTER` triggers) and the trigger fires **once per statement**, not once per row.
2. `inserted` and `deleted` contain *all* affected rows (zero, one or many). Built from the row-versioning store in tempdb.
3. The trigger runs inside the statement's transaction. A `ROLLBACK` in the trigger undoes the statement *and* the whole outer transaction.
4. An `INSTEAD OF` trigger runs *instead of* the change; it must perform the modification itself. Used to make complex views updatable.

**Dynamic SQL, safely:**
```sql
DECLARE @sql nvarchar(max) = N'
SELECT orderid, orderdate FROM ' + QUOTENAME(@schema) + N'.' + QUOTENAME(@table) + N'
WHERE custid = @custid;';
EXEC sys.sp_executesql @sql, N'@custid int', @custid = @custid;
```
Identifiers (which can't be parameters) are wrapped with `QUOTENAME`; values are passed as real parameters, so they can never become code, and the plan is reused for every `@custid`.

### 4. Worked example
```sql
CREATE OR ALTER PROCEDURE dbo.TransferStock
    @fromWarehouse int, @toWarehouse int, @productid int, @qty int
AS
BEGIN
    SET NOCOUNT ON;
    SET XACT_ABORT ON;

    IF @qty <= 0
        THROW 50001, 'Quantity must be positive.', 1;

    BEGIN TRY
        BEGIN TRAN;

        UPDATE dbo.Stock
        SET qty -= @qty
        WHERE warehouseid = @fromWarehouse AND productid = @productid AND qty >= @qty;

        IF @@ROWCOUNT = 0
            THROW 50002, 'Insufficient stock or product not found.', 1;

        UPDATE dbo.Stock
        SET qty += @qty
        WHERE warehouseid = @toWarehouse AND productid = @productid;

        IF @@ROWCOUNT = 0
            INSERT dbo.Stock (warehouseid, productid, qty) VALUES (@toWarehouse, @productid, @qty);

        COMMIT;
    END TRY
    BEGIN CATCH
        IF @@TRANCOUNT > 0 ROLLBACK;

        INSERT dbo.ErrorLog (errnum, errmsg, errproc, errline, logged_at)
        VALUES (ERROR_NUMBER(), ERROR_MESSAGE(), ERROR_PROCEDURE(), ERROR_LINE(), SYSDATETIME());

        THROW;      -- re-raise the original error to the caller
    END CATCH;
END;
```
Note the order in CATCH: roll back *first*, then log. Logging inside a doomed transaction fails, and logging before the rollback would be rolled back too.

**A set-based audit trigger** (handles multi-row statements):
```sql
CREATE OR ALTER TRIGGER dbo.trg_Stock_Audit ON dbo.Stock
AFTER UPDATE
AS
BEGIN
    SET NOCOUNT ON;
    IF NOT EXISTS (SELECT * FROM inserted) RETURN;          -- statement touched 0 rows

    INSERT dbo.StockAudit (warehouseid, productid, oldqty, newqty, changed_at, changed_by)
    SELECT i.warehouseid, i.productid, d.qty, i.qty, SYSDATETIME(), ORIGINAL_LOGIN()
    FROM inserted AS i
    JOIN deleted  AS d ON d.warehouseid = i.warehouseid AND d.productid = i.productid
    WHERE i.qty <> d.qty;
END;
```
A trigger written as `SELECT @id = id FROM inserted` handles only one arbitrary row of a multi-row update: a classic bug.

### 5. Under the hood
- **Scalar UDFs** historically ran once per row, in a separate context, blocked parallelism for the whole query, and were invisible in plans and I/O statistics. SQL Server 2019 (compat 150) added **scalar UDF inlining**, which rewrites eligible UDFs into the query as expressions. Many UDFs are not eligible (e.g. those using `GETDATE()`, table variables, or certain constructs); check `sys.sql_modules.is_inlineable`.
- **Multi-statement TVFs** return a table variable; fixed cardinality estimates (1 row, or 100 in compat 120–130) until *interleaved execution* in SQL 2017+. Prefer inline TVFs.
- **Table variables** have no column statistics. Before SQL 2019 the optimizer assumed 1 row; SQL 2019 *deferred compilation* uses the actual row count at first execution. Temp tables have full statistics and can cause recompiles as data changes.
- **Procedure plan caching** and **parameter sniffing**: the first call's parameter values shape the cached plan (see the Query Processing block). Options: `OPTION (RECOMPILE)`, `OPTIMIZE FOR`, Query Store hints, SQL 2022 PSP optimization.
- **Temp table caching**: temp tables created in procedures can be cached and reused across calls (reducing tempdb allocation contention) unless you run DDL on them after creation (e.g. `CREATE INDEX` separately instead of inline index definitions).
- **Trigger cost**: `inserted`/`deleted` are built from the version store, adding tempdb load. Nested triggers (up to 32 levels) and recursive triggers can multiply work. Triggers run inside the caller's transaction, extending lock duration.

### 6. Performance and design implications
- Keep business data changes set-based; use cursors only for admin work where per-row processing is the point (e.g. iterate databases). If you must, use `LOCAL FAST_FORWARD`.
- Prefer inline TVFs over scalar UDFs and multi-statement TVFs in queries over many rows.
- Use temp tables for intermediate results larger than a few hundred rows or that need good estimates; table variables for tiny sets.
- Keep triggers short and set-based; never call external resources (email, linked servers) from triggers.
- Always parameterize dynamic SQL; it's both a security and a plan-cache issue.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SET XACT_ABORT ON` | Runtime error dooms and rolls back the transaction | Every procedure with transactions |
| `THROW [num, msg, state]` | Raise / re-raise errors (always terminates batch flow) | Modern error handling |
| `XACT_STATE()` | Is the transaction committable? | In CATCH |
| `sp_executesql` | Parameterized dynamic SQL | Dynamic search, dynamic pivots |
| `QUOTENAME()` | Safely delimits identifiers | Table/column names in dynamic SQL |
| `CREATE OR ALTER` | Create or replace an object | Deployment scripts |
| `WITH RECOMPILE` / `OPTION (RECOMPILE)` | Fresh plan per call / per statement | Sniffing-sensitive code |
| `sys.sql_modules.is_inlineable` | Scalar UDF eligible for inlining | UDF review |
| `TRIGGER_NESTLEVEL()` | Depth of trigger nesting | Prevent recursion |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Stored procedures: security (EXECUTE only), plan reuse, one round trip | Logic that needs frequent app-side changes or complex string work |
| Inline TVF: reusable parameterized query | — |
| Scalar UDF: simple logic, SQL 2019+ inlinable | Pre-2019 or non-inlinable, called per row on large sets |
| AFTER trigger: audit, cross-table rules | Business workflows, external calls, heavy logic |
| INSTEAD OF trigger: updatable complex views | Hidden side effects that surprise developers |
| Temp table: larger sets, need stats/indexes | Tiny sets in hot procedures → table variable |
| Dynamic SQL: optional filters, dynamic object names | Concatenating user values |

### 9. Common mistakes
- Trigger written for one row → wrong results on multi-row statements → join `inserted` and `deleted`.
- Logging in CATCH before ROLLBACK → log entry lost or error 3930 → roll back first.
- Swallowing errors in CATCH without `THROW` → caller thinks it worked → re-raise.
- Concatenating parameters into dynamic SQL → SQL injection, plan cache bloat → `sp_executesql` + `QUOTENAME`.
- `SELECT @var = col FROM t` returning many rows → silently keeps one value → use `SET` with a subquery or `TOP (1) … ORDER BY`.
- No `SET XACT_ABORT ON` → client timeout leaves an open transaction holding locks.
- Scalar UDF in a WHERE clause over millions of rows → serial, per-row execution → inline it as an iTVF or expression.

### 10. 🎯 Interview kit
**30-second answer:** SQL Server lets you store logic as procedures, functions and triggers, and write procedural T-SQL with variables, flow control, temp tables and dynamic SQL. The key practices: SET NOCOUNT and XACT_ABORT ON, TRY/CATCH with rollback-then-THROW, parameterized sp_executesql, set-based triggers that handle many rows, and inline TVFs over scalar or multi-statement functions for performance.

**Structure a longer answer:** objects and when to use each → error-handling pattern → performance traps (scalar UDF, mTVF, table variables) and SQL 2017/2019 fixes → security (injection, ownership chaining) → a procedure you hardened.

**Likely follow-ups:**
- *Temp table vs. table variable?* → stats, scope, recompiles, rollback behavior.
- *Why are scalar UDFs slow?* → per-row execution, no parallelism; inlining in 2019.
- *What does XACT_ABORT do?* → runtime errors roll back the whole transaction; covers client timeouts.
- *How does a trigger see changed rows?* → inserted/deleted pseudo-tables, once per statement.

**Signals interviewers look for:** multi-row trigger awareness; correct CATCH order; injection-safe dynamic SQL; knowledge of version-specific fixes.

### 11. Check yourself
- [ ] Write a procedure with the TRY/CATCH + XACT_ABORT pattern and force an error to test it.
- [ ] Update 3 rows through a single-row-style trigger and see the bug; fix it.
- [ ] Turn a concatenated dynamic query into `sp_executesql` with parameters.
- [ ] Compare a scalar UDF vs. inline expression on 1M rows at compat 140 and 150.
- [ ] List four differences between temp tables and table variables.
<!-- explainer:end -->

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Error Handling in Stored Procedures
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 7: Dynamic SQL, Triggers, and Table-Valued Functions (TVFs) › Introduction to Dynamic SQL, Triggers, and TVFs
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Programmable objects
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — User-Defined Data Types › Triggers
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Error Handling with TRY...CATCH
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 7: Dynamic SQL, Triggers, and Table-Valued Functions (TVFs) › Structure
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 7: Dynamic SQL, Triggers, and Table-Valued Functions (TVFs) › Conclusion
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 7: Dynamic SQL, Triggers, and Table-Valued Functions (TVFs) › Using AFTER and INSTEAD OF Triggers
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Functions Available in CATCH Block
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Best Practices in Error Handling

