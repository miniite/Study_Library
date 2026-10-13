---
id: "01_Databases_and_SQL/Database_Objects/Stored_Procedures/Stored_Procedure_Fundamentals"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Objects/Stored_Procedures"
tools: ["SQL_Server", "MySQL"]
---

# Stored Procedure Fundamentals

**L4 Advanced** · [↑ Stored Procedures](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A stored procedure is a named, saved block of SQL plus procedural logic that lives and runs inside the database — callable by name with parameters, able to modify data, control transactions and handle errors, and usable as a security and reuse boundary between applications and tables.

### 1. The big picture
Without stored procedures, every application that needs "transfer money" or "place an order" re-implements the same several SQL statements, each needing the right checks, transaction handling and error behaviour. Risks: inconsistent rules between applications, many network round trips, direct table permissions everywhere, and changes that require redeploying every client.

A **stored procedure** packages those steps as one unit in the database. Callers send the procedure name and parameters. Benefits:
- **Reuse and consistency** — one definition of the business operation.
- **Security** — callers get `EXECUTE` on the procedure without permissions on the tables (ownership chaining in SQL Server, `DEFINER` rights in MySQL, `SECURITY DEFINER` in PostgreSQL). The procedure also removes the temptation to concatenate SQL in the app.
- **Fewer round trips** — logic runs next to the data; only the final result crosses the network.
- **Plan reuse** — statements are compiled once and cached, then reused with different parameter values.
- **Isolation from schema change** — internals can change while the signature stays stable.
- **Unit of work** — a whole multi-statement operation is atomic under one transaction.

The costs: logic spread between app and database, vendor-specific languages (T-SQL, PL/SQL, PL/pgSQL, MySQL procedural SQL), harder testing/debugging/versioning, and scaling a database CPU is more expensive than scaling app servers. "Don't turn them into mini-applications" is the standard advice.

### 2. Core concepts and vocabulary
- **Stored procedure (proc, sproc)** — named routine stored in the database, executed with `EXEC`/`CALL`.
- **Function (UDF)** — returns a value or table and can be used inside queries; procedures can't be used in `SELECT`.
- **Trigger** — routine fired automatically by DML/DDL events; procedures are called explicitly.
- **Batch / body** — the statements between `AS BEGIN … END`.
- **Control-of-flow** — `IF/ELSE`, `WHILE`, `CASE`, `RETURN`, `GOTO`.
- **Local variable** — `DECLARE @x INT; SET @x = …` (T-SQL), `DECLARE x INT;` (MySQL/PL/SQL).
- **Result set** — rows streamed to the caller by a `SELECT` in the body; a procedure can return several.
- **Output parameter / return code** — scalar values to the caller (see Procedure Parameters).
- **Execution context / ownership chaining** — whose permissions apply when the procedure touches objects.
- **System catalog** — metadata tables (`sys.sql_modules`, `information_schema.routines`, `pg_proc`, `ALL_SOURCE`) where the definition text is stored.
- **Compile / plan cache** — the optimizer's plan per statement, cached after first execution.
- **`CREATE OR ALTER`** — replace the definition in place keeping permissions (SQL Server 2016 SP1+).

### 3. How it works, step by step
**Create**
1. `CREATE PROCEDURE schema.name (params) AS BEGIN … END` is parsed and syntax-checked; object names may be resolved later (**deferred name resolution** in SQL Server — a procedure can reference a table that doesn't exist yet).
2. The definition text is stored in the catalog (e.g. `sys.sql_modules.definition`), the procedure is registered in `sys.objects`; no plan exists yet.

**Execute (first call)**
1. Permission check (`EXECUTE` on the procedure).
2. The statements are compiled with the first call's parameter values, producing one plan per statement; plans go to the plan cache.
3. The batch runs: assignments, control flow, DML, result sets streamed to the client.
4. Errors raise exceptions (caught by `TRY…CATCH`/handlers) or abort the batch depending on severity and `XACT_ABORT`.
5. Locals disappear, OUTPUT values and return code go back; open transaction state is verified.

**Subsequent calls** reuse cached plans unless evicted, recompiled (schema change, stale statistics, `RECOMPILE`) or a SET option differs.

**Typical body skeleton (T-SQL)**
```sql
CREATE OR ALTER PROCEDURE dbo.usp_TransferFunds
    @FromAcct INT, @ToAcct INT, @Amount DECIMAL(12,2)
AS
BEGIN
    SET NOCOUNT ON;
    SET XACT_ABORT ON;                    -- any runtime error rolls back the transaction

    IF @Amount <= 0 THROW 50010, 'Amount must be positive.', 1;

    BEGIN TRY
        BEGIN TRAN;
            UPDATE dbo.Account SET Balance -= @Amount
            WHERE AccountID = @FromAcct AND Balance >= @Amount;
            IF @@ROWCOUNT = 0 THROW 50011, 'Insufficient funds or unknown account.', 1;

            UPDATE dbo.Account SET Balance += @Amount WHERE AccountID = @ToAcct;
            IF @@ROWCOUNT = 0 THROW 50012, 'Unknown target account.', 1;

            INSERT dbo.AccountLedger(FromAcct, ToAcct, Amount, CreatedAt)
            VALUES (@FromAcct, @ToAcct, @Amount, SYSUTCDATETIME());
        COMMIT;
    END TRY
    BEGIN CATCH
        IF @@TRANCOUNT > 0 ROLLBACK;
        THROW;                            -- re-raise to the caller
    END CATCH
END;
```

### 4. Worked example
```sql
-- Create and call
EXEC dbo.usp_TransferFunds @FromAcct = 1001, @ToAcct = 2002, @Amount = 250.00;

-- Where is it stored? What does it depend on?
SELECT o.name, m.definition
FROM sys.sql_modules m JOIN sys.objects o ON o.object_id = m.object_id
WHERE o.name = 'usp_TransferFunds';

EXEC sp_helptext 'dbo.usp_TransferFunds';
SELECT * FROM sys.dm_sql_referenced_entities('dbo.usp_TransferFunds', 'OBJECT');

-- List procedures in a database
SELECT SCHEMA_NAME(schema_id) AS sch, name, create_date, modify_date
FROM sys.procedures ORDER BY modify_date DESC;

-- Grant execute only; the caller has no table permissions
GRANT EXECUTE ON dbo.usp_TransferFunds TO app_role;
```
Calling from an application (Python/psycopg2 for PostgreSQL 11+ procedures; JDBC `CallableStatement` is analogous):
```python
cur.execute("CALL transfer_funds(%s, %s, %s)", (1001, 2002, 250.00))
```
```java
CallableStatement cs = conn.prepareCall("{call usp_TransferFunds(?,?,?)}");
cs.setInt(1, 1001); cs.setInt(2, 2002); cs.setBigDecimal(3, new BigDecimal("250.00"));
cs.execute();
```
Walk-through of the transfer: the debit has a `Balance >= @Amount` predicate and a `@@ROWCOUNT` check, so insufficient funds is detected atomically without a race (no separate "read balance then update"). Everything sits inside one transaction; the `CATCH` block rolls back and re-throws so the caller sees the original error and number. `XACT_ABORT ON` ensures that even errors `TRY…CATCH` doesn't catch (like some client disconnects/timeouts) roll back instead of leaving a transaction open.

### 5. Under the hood
- **Where and what is stored.** The *text* is in the catalog (SQL Server: `sys.sql_modules`; PostgreSQL: `pg_proc.prosrc`; MySQL: `mysql.proc` / `information_schema.ROUTINES`; Oracle: `ALL_SOURCE`, plus compiled p-code). The *plan* isn't stored persistently; it's built on first use (PL/SQL is compiled to p-code and optionally native code at creation).
- **"Precompiled" nuance.** The commonly repeated claim is that procedures are precompiled; in SQL Server they compile on first execution and cache the plan per statement, and parameterised ad-hoc/prepared statements can get the same reuse. The real benefit is *predictable parameterisation and stable text*, not magic speed — Fritchey's performance chapters list network-traffic reduction, logic near the data, and plan-caching effectiveness as the actual wins.
- **Security model.** In SQL Server, ownership chaining lets a caller with `EXECUTE` on the proc access tables owned by the same owner without table grants (breaks with dynamic SQL, cross-database chains unless enabled). MySQL `SQL SECURITY DEFINER|INVOKER` and PostgreSQL `SECURITY DEFINER|INVOKER` choose whose rights apply; a `DEFINER` routine is powerful and a classic privilege-escalation risk if it builds SQL from input or doesn't pin `search_path`.
- **Transactions.** Procedure calls don't start a transaction by themselves (T-SQL autocommit applies per statement). Nested `BEGIN TRAN` increments `@@TRANCOUNT`; only the outermost `COMMIT` commits, and any `ROLLBACK` rolls back everything — pay attention when procedures call each other. PostgreSQL procedures (not functions) may `COMMIT`/`ROLLBACK` internally; functions can't. MySQL procedures can't contain DDL implicit-commit surprises (DDL commits).
- **Result sets.** A T-SQL procedure can emit multiple `SELECT` result sets — drivers expose them sequentially (`NextResultSet`, JDBC `getMoreResults`). MySQL supports multiple result sets via the protocol; PostgreSQL procedures return via `OUT` parameters, `refcursor`s or temp tables, and *functions* `RETURNS SETOF`/`TABLE`.
- **Temp tables and recompilation.** Creating temp tables inside a procedure can cause statement-level recompiles (deferred compile); passing the data in via TVP or creating the table early helps.
- **Dependencies.** Renaming a table won't update referencing procedures; use `sys.sql_expression_dependencies`/`sp_depends` to detect, and `sp_refreshsqlmodule` after schema change. Source control (database projects, migrations) is mandatory; editing in prod is how drift happens.
- **Metadata and naming.** Avoid the `sp_` prefix in SQL Server: it first searches `master`, adding lookup cost and risking name collisions. Use schema-qualified names (`dbo.usp_…`) so plan cache entries aren't per-user.

### 6. Performance and design implications
- Use `SET NOCOUNT ON` to stop "(n rows affected)" messages for every statement; reduces chatter and avoids breaking some clients (ADO.NET adapters).
- Schema-qualify objects, avoid `SELECT *`, and keep statements set-based; keep procedures focused (single responsibility) rather than 2,000-line monoliths. Break out reusable steps into sub-procedures *when it improves clarity*, not at the cost of excessive nesting.
- Keep transactions short: do validation first, lock late, commit early; handle deadlock retries (error 1205) in the caller or with retry logic.
- Avoid scalar UDFs in hot paths (pre-2019 SQL Server executes them row by row) — inline the logic or use inline table-valued functions.
- Return only what's needed: pagination inside the procedure rather than sending 100,000 rows to display 20.
- Measure: `sys.dm_exec_procedure_stats` (execution_count, total_worker_time, total_logical_reads), Query Store runtime stats, `SET STATISTICS IO, TIME ON`, and per-statement plans.
- Plan for operations: documentation header (purpose, author, parameters, changes), consistent naming (`usp_<Verb><Noun>`), tests (tSQLt, pgTAP, utPLSQL), and code review like any application code.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE [OR ALTER] PROCEDURE` | Create/replace procedure | Deployments |
| `EXEC proc @p = v` / `CALL proc(v)` | Execute | Calling |
| `DROP PROCEDURE [IF EXISTS]` | Remove | Retiring |
| `SET NOCOUNT ON`, `SET XACT_ABORT ON` | Suppress row-count messages; auto-rollback on error | Top of nearly every T-SQL proc |
| `BEGIN TRY … END TRY BEGIN CATCH … THROW` | Structured error handling | Any modifying procedure |
| `sp_helptext`, `OBJECT_DEFINITION()`, `SHOW CREATE PROCEDURE`, `\sf proc` | See source | Review, debugging |
| `sys.procedures`, `information_schema.routines`, `pg_proc` | List procedures | Inventory |
| `GRANT EXECUTE ON … TO …` | Permission to run | Security boundary |
| `WITH RECOMPILE`, `OPTION (RECOMPILE)`, `sp_recompile` | Force fresh plans | Sniffing/outlier issues |
| `WITH EXECUTE AS OWNER`, `SECURITY DEFINER` | Context switch | Controlled privilege elevation |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| **Stored procedure** — multi-step transactional operation, strict security boundary, performance-critical set logic, many callers | Simple one-table CRUD already handled by ORM; heavy non-data business logic |
| **Application-layer logic** — complex business rules, easier testing, scale-out compute | You need atomic multi-statement semantics near data or reduce chattiness |
| **Function** — need result inside a SELECT/JOIN | Needs side effects/transactions (functions can't modify data in most engines) |
| **Trigger** — enforce invariants regardless of caller, audit | Hidden logic, performance surprises; keep light and set-based |
| **View** — reusable read-only projection | Parameters, side effects, multiple result sets |
| **Many small procedures** — reuse and testability | Call-chain overhead and transaction nesting confusion |
| **One big procedure** — one cohesive workflow | Unreadable, impossible to test or reuse pieces |

### 9. Common mistakes
- **No error handling or transaction in multi-statement procedures** → partial updates → `TRY/CATCH` + `XACT_ABORT` + explicit transaction.
- **`sp_` prefix** (SQL Server) → master lookup, collisions → use `usp_`/`proc_`.
- **Unqualified object names** → cache misses, wrong schema resolution → `dbo.` prefix.
- **Business logic sprawl and 1000-line procs** → untestable → split by responsibility, use set-based statements.
- **Trusting the caller for permissions** → missing validation → check input in the procedure.
- **Dynamic SQL with concatenation in DEFINER/owner context** → injection + privilege escalation → parameterised `sp_executesql`, `QUOTENAME`.
- **Assuming rows-affected messages are harmless** → client confusion and chatter → `SET NOCOUNT ON`.
- **Editing procedures directly in production** → drift → source-controlled deployments.
- **Nested transactions with unmatched rollbacks** → "transaction count mismatch" errors → centralise transaction ownership.
- **Treating "precompiled" as a guarantee of speed** → ignoring sniffing/recompiles → measure.

### 10. 🎯 Interview kit
**30-second answer:** "A stored procedure is a named, saved block of SQL and procedural logic stored in the database and executed by name with parameters. It lets you encapsulate a multi-step operation in one transaction, reduce round trips, reuse it across applications, and grant `EXECUTE` without table access. The trade-offs are vendor-specific language, harder testing/versioning, and putting logic on the scarce database CPU."

**Structure a longer answer:** what/why (encapsulation, security, performance) → how it executes (catalog, compile, cache) → anatomy (parameters, variables, flow control, error handling, transactions) → alternatives (functions, triggers, views, app logic) → pitfalls and best practices.

**Likely follow-ups:**
- *Procedure vs function?* → Functions return values and are usable in queries (and must be side-effect free in most engines); procedures can modify data, return multiple result sets and manage transactions.
- *Procedure vs trigger?* → Explicit call vs event-driven.
- *Where is the definition stored?* → system catalog (`sys.sql_modules`, `pg_proc`, `information_schema.routines`).
- *Can a procedure return a value / multiple result sets?* → return code, OUT parameters, multiple `SELECT`s (T-SQL/MySQL) or refcursors/`SETOF` (PostgreSQL).
- *Can procedures call each other?* → Yes; `EXEC`/`CALL`; permissions and transaction nesting apply.
- *Advantages over inline SQL?* → reuse, security, fewer round trips, plan stability, encapsulation.
- *How do you pass lists / handle errors / permissions?* → TVP/array; TRY…CATCH/SIGNAL/RAISE; `GRANT EXECUTE`.
- *When would you not use them?* → simple CRUD with an ORM, portability requirements, heavy non-data logic.

**Signals interviewers look for:** concrete knowledge of execution and security model rather than slogans, error/transaction handling, and balanced opinions on where logic belongs.

### 11. Check yourself
- [ ] Write a procedure that inserts a customer given name and email with validation and error handling.
- [ ] Explain ownership chaining and how it gives table access without table permissions.
- [ ] Show how to list all procedures and view the source in your engine.
- [ ] Contrast procedure, function, trigger and view in one table.
- [ ] Describe what happens on the first versus the tenth call to a procedure in terms of plans.
<!-- explainer:end -->

## Implementations (tools)

- [SQL Server](<../../../TOOLS/SQL_Server/_index.md>)
- [MySQL](<../../../TOOLS/MySQL/_index.md>)

## Practice questions

- [Basic] [How do you declare and use local variables inside a stored procedure?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Stored_Procedures/Q-41841_How_do_you_declare_and_use_local_variables_inside_a_stored_p.md>)
- [Basic] [Where is the definition of a stored procedure stored in the database?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Stored_Procedures/Q-41890_Where_is_the_definition_of_a_stored_procedure_stored_in_the_.md>)
- [Intermediate] [Can a stored procedure return a value like a function does? Explain how.](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Stored_Procedures/Q-41819_Can_a_stored_procedure_return_a_value_like_a_function_does_E.md>)
- [Intermediate] [Write a simple stored procedure that inserts a new customer record given name and email as inputs.](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Stored_Procedures/Q-41903_Write_a_simple_stored_procedure_that_inserts_a_new_customer_.md>)
- [Intermediate] [Can you use a SELECT statement inside a stored procedure to fill a temporary table? How?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Stored_Procedures/Q-41922_Can_you_use_a_SELECT_statement_inside_a_stored_procedure_to_.md>)
- [Intermediate] [How would you call a stored procedure from a Java or Python application using JDBC or a driver?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Stored_Procedures/Q-41944_How_would_you_call_a_stored_procedure_from_a_Java_or_Python_.md>)
- [Deep] [What is a DECLARE HANDLER statement in MySQL and what does CONTINUE versus EXIT mean?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Stored_Procedures/Q-41877_What_is_a_DECLARE_HANDLER_statement_in_MySQL_and_what_does_C.md>)
- [Deep] [How do you build and execute a dynamic SQL statement using PREPARE and EXECUTE in a MySQL procedure?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Stored_Procedures/Q-41901_How_do_you_build_and_execute_a_dynamic_SQL_statement_using_P.md>)

## Related concepts

- [Dynamic SQL](<../../Database_Internals/DBMS_Concepts/Dynamic_SQL.md>) *(DBMS Concepts)*
- [Functions in SQL vs PL/SQL](<../../../TOOLS/Oracle/PL-SQL/Functions_in_SQL_vs_PLSQL.md>) *(PL-SQL)*
- [MERGE and Upserts](<../../../TOOLS/SQL_Server/SQL_Server_Platform/MERGE_and_Upserts.md>) *(SQL Server Platform)*
- [Stored Procedure Performance and Design](<Stored_Procedure_Performance_and_Design.md>) *(Stored Procedures)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Stored Procedures in SQL Server
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Creating Stored Procedures
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Parameters in Stored Procedures
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Returning Values from Procedures
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Stored Procedures
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Stored Procedures › Saving the Results of a Stored Procedure in a Table
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Stored Procedures › Using a Logic in Stored Procedures
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Expanding on Data Type Concepts › Using Stored Procedures to Manage Hierarchical Data
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 6 Using The Query Store For Query Performance And Execution Plans
- *Azure Data Fundamentals* — Michael John Pena — Relational Data Concepts › Stored Procedures: Creating Reusable Code
- *Designing Data-Intensive Applications* — Martin Kleppmann — Serializability › Encapsulating transactions in stored procedures
- *Designing Data-Intensive Applications* — Martin Kleppmann — Serializability › Pros and cons of stored procedures
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Development standards › 2.2 #1 Nondescriptive object names
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Development standards › 2.4 #3 The dreaded sp\ prefix
- *SQL Server Tutorial* (YouTube series) — Part 18: Stored procedures in sql server ([watch](https://www.youtube.com/watch?v=Qu3E-oncF3g))
- *SQL Server Tutorial* (YouTube series) — Part 19: Stored procedures with output parameters ([watch](https://www.youtube.com/watch?v=bldBshxuhMk))
- *SQL Server Tutorial* (YouTube series) — Part 20: Stored procedure output parameters or return values ([watch](https://www.youtube.com/watch?v=st8RnNg_LLA))
- *SQL Server Tutorial* (YouTube series) — Part 100: Send datatable as parameter to stored procedure ([watch](https://www.youtube.com/watch?v=P3uBWp0qvlo))

