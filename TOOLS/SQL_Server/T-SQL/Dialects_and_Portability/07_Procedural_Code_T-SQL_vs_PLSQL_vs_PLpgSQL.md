---
id: "TOOLS/SQL_Server/T-SQL/Dialects_and_Portability/07_Procedural_Code_T-SQL_vs_PLSQL_vs_PLpgSQL"
level: 5
type: "concept"
topic: "TOOLS/SQL_Server/T-SQL/Dialects_and_Portability"
tools: ["SQL_Server"]
---

# 7 · Procedural Code: T-SQL vs PL/SQL vs PL/pgSQL

**L5** · [↑ Series index](<_index.md>) · Prev: [Transactions and Concurrency Across Engines](<06_Transactions_and_Concurrency_Across_Engines.md>) · Next: [T-SQL Across the Microsoft Family](<08_T-SQL_Across_the_Microsoft_Family.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** The *query* half of SQL is nearly portable; the *procedural* half — variables, control flow, error handling, cursors, dynamic SQL, procedures and triggers — is where each vendor built its own language, so this is where porting costs are highest.

### 1. Two languages in one name

Every major engine has a SQL layer (declarative, set-based) and a procedural layer (imperative, row-at-a-time control flow):

| Engine | Declarative | Procedural | Where procedural code runs |
|---|---|---|---|
| SQL Server | T-SQL queries | **T-SQL** control flow (same dialect!) | Batches, procedures, functions, triggers |
| Oracle | Oracle SQL | **PL/SQL** (block-structured, Ada-like) | Anonymous blocks, procedures, functions, packages, triggers |
| PostgreSQL | PostgreSQL SQL | **PL/pgSQL** (also PL/Python, PL/Perl…) | Functions, procedures (11+), `DO` blocks, triggers |
| MySQL | MySQL SQL | stored-routine syntax (SQL/PSM-like) | Procedures, functions, triggers, events |
| Db2 | Db2 SQL | **SQL PL** | Procedures, functions |
| Snowflake | Snowflake SQL | **Snowflake Scripting**, JavaScript/Python/Java procs | Procedures, anonymous blocks |

A key T-SQL distinction: **there is no clean separation** — you can sprinkle `IF`, `WHILE` and variables between ordinary statements in a batch with no `BEGIN…END` wrapper or procedure, which is why ad-hoc scripts and ADF script blocks can use them. Oracle and PostgreSQL require a block (`BEGIN … END;` / `DO $$ … $$`) for any procedural logic.

### 2. Side-by-side reference

| Concept | T-SQL | PL/SQL (Oracle) | PL/pgSQL (PostgreSQL) |
|---|---|---|---|
| Declare variable | `DECLARE @n int = 5;` | `n NUMBER := 5;` (in `DECLARE` section) | `n integer := 5;` (in `DECLARE` section) |
| Assign | `SET @n = 6;` / `SELECT @n = col FROM t` | `n := 6;` / `SELECT col INTO n FROM t;` | `n := 6;` / `SELECT col INTO n FROM t;` |
| Branch | `IF … BEGIN … END ELSE BEGIN … END` | `IF … THEN … ELSIF … ELSE … END IF;` | `IF … THEN … ELSIF … ELSE … END IF;` |
| Loop | `WHILE cond BEGIN … END` (only loop; `BREAK`/`CONTINUE`) | `LOOP/WHILE/FOR … END LOOP;` `EXIT`, `CONTINUE` | `LOOP/WHILE/FOR … END LOOP;` |
| Case | `CASE` expression only (no `CASE` statement) | `CASE` statement and expression | `CASE` statement and expression |
| Error handling | `BEGIN TRY … END TRY BEGIN CATCH … END CATCH` + `ERROR_NUMBER()`, `ERROR_MESSAGE()` | `EXCEPTION WHEN NO_DATA_FOUND THEN … WHEN OTHERS THEN …` | `EXCEPTION WHEN unique_violation THEN … WHEN OTHERS THEN …` |
| Raise | `THROW 50001,'msg',1;` / `RAISERROR('msg',16,1)` | `RAISE_APPLICATION_ERROR(-20001,'msg');` | `RAISE EXCEPTION 'msg';` |
| Affected rows | `@@ROWCOUNT` (fragile — changes after next statement) | `SQL%ROWCOUNT` | `GET DIAGNOSTICS n = ROW_COUNT;` |
| Last identity | `SCOPE_IDENTITY()`, `OUTPUT` | `RETURNING … INTO` | `RETURNING … INTO` |
| Cursor | `DECLARE c CURSOR FOR …; OPEN; FETCH NEXT … INTO …; WHILE @@FETCH_STATUS = 0` | `CURSOR c IS …; FOR r IN c LOOP … END LOOP;` | `FOR r IN SELECT … LOOP … END LOOP;` |
| Dynamic SQL | `EXEC sp_executesql @sql, N'@p int', @p = 1;` / `EXEC(@sql)` | `EXECUTE IMMEDIATE sql USING p;` | `EXECUTE format(...) USING p;` |
| Return result set | Any `SELECT` inside a procedure goes to the client automatically | `OUT SYS_REFCURSOR` or pipelined function | `RETURNS TABLE / SETOF` function or `refcursor` |
| Output parameters | `@x int OUTPUT` | `x OUT NUMBER` | `x OUT integer` |
| Functions | scalar, inline TVF, multi-statement TVF | functions, pipelined functions | functions (`RETURNS`) |
| Packages / modules | none (use schemas + naming) | **packages** | schemas / extensions |
| Table-valued inputs | `READONLY` table-valued parameter (TVP) | collections / `TABLE OF` types | arrays, `unnest`, composite types |
| Triggers | `AFTER`/`INSTEAD OF`, **statement-level**, `inserted`/`deleted` pseudo-tables | `BEFORE/AFTER`, **row or statement**, `:NEW`/`:OLD` | `BEFORE/AFTER/INSTEAD OF`, row or statement, `NEW`/`OLD` records |
| Temp structure | `#temp`, `@table` | global temporary tables, collections | `TEMP TABLE`, arrays |
| Debug output | `PRINT`, `RAISERROR … WITH NOWAIT` | `DBMS_OUTPUT.PUT_LINE` | `RAISE NOTICE` |
| Transactions in routines | allowed (`BEGIN TRAN` inside procs; counters) | allowed (`COMMIT` inside procedures; autonomous transactions with pragma) | `COMMIT` allowed only in procedures (11+), not functions |

### 3. Concepts unique to T-SQL (and their consequences)

#### 3.1 Batches and the `GO` separator
A *batch* is the compilation and execution unit. Variables **do not survive** a batch boundary:

```sql
DECLARE @x int = 1;
GO                      -- client directive: ends the batch
SELECT @x;              -- Msg 137: Must declare the scalar variable "@x"
```
In tools that never see `GO` (ADF script blocks, `sp_executesql`, `pyodbc`), you write one batch only.

#### 3.2 Result sets come out of procedures implicitly
A procedure that executes `SELECT` statements *returns them to the caller* — often accidentally (a stray `SELECT` for debugging breaks the Lookup activity that expects one result set). Oracle/PG require an explicit cursor or function return.

#### 3.3 Row-by-row vs set-based thinking
T-SQL's cursors and `WHILE` loops are **interpreted** statement by statement and the optimizer plans each embedded statement separately. A loop that runs `INSERT` per row is often 10–1000× slower than one `INSERT … SELECT`. PL/SQL and PL/pgSQL have the same law (context switches between engines), but PL/SQL's bulk-collect/`FORALL` are *named* optimisations; T-SQL's equivalent is "just write set-based SQL". See [Row-by-Row Processing](<../../04_Query_Processing/Row_by_Row_Processing/_index.md>).

#### 3.4 `@@ROWCOUNT` and `@@ERROR` are volatile
They describe the *immediately preceding* statement and reset on the next statement — including `IF`, `SET`, even `PRINT`.

```sql
UPDATE dbo.T SET c = 1 WHERE id = 5;
IF @@ROWCOUNT = 0   -- OK: first statement after the UPDATE
    PRINT 'no row';
SELECT @@ROWCOUNT;  -- always 0 (reset by the IF)
```
Capture them immediately: `DECLARE @rc int = @@ROWCOUNT;`.

#### 3.5 Variables and scope
- `@var` scope is the batch (or the procedure body). Variables are *not* block-scoped within `BEGIN…END`.
- `SELECT @v = col FROM t` with multiple rows assigns the **last** row silently; `SET @v = (SELECT col FROM t)` errors when a subquery returns more than one row (Msg 512).

#### 3.6 Table variables and temp tables
Table variables are batch-scoped and have no statistics (cardinality guess of 1 row before 2019's deferred compilation); `#temp` tables have stats and are visible in nested procedures. See [Temp Tables vs Table Variables](<../Temp_Tables_vs_Table_Variables.md>).

#### 3.7 Error handling nuances
- `TRY…CATCH` catches errors of severity > 10 that do not terminate the connection; **compile errors in the same scope are not caught** (e.g., calling a nonexistent column in the same batch).
- `THROW` (2012+) preserves the original error number and message; `RAISERROR` does not.
- A transaction in a doomed state (`XACT_STATE() = -1`) can only be rolled back.

### 4. A worked translation — the same procedure in three languages

Task: *insert an order line; if the product is out of stock raise an error; return the new id.*

```sql
-- T-SQL
CREATE OR ALTER PROCEDURE dbo.AddOrderLine
    @OrderID int, @ProductID int, @Qty int, @LineID int OUTPUT
AS
BEGIN
    SET NOCOUNT ON; SET XACT_ABORT ON;
    BEGIN TRY
        BEGIN TRAN;
        UPDATE dbo.Product SET Stock -= @Qty WHERE ProductID = @ProductID AND Stock >= @Qty;
        IF @@ROWCOUNT = 0 THROW 50001, 'Out of stock', 1;
        INSERT dbo.OrderLine(OrderID, ProductID, Qty) VALUES (@OrderID, @ProductID, @Qty);
        SET @LineID = SCOPE_IDENTITY();
        COMMIT;
    END TRY
    BEGIN CATCH
        IF @@TRANCOUNT > 0 ROLLBACK;
        THROW;
    END CATCH;
END;
```
```sql
-- PL/SQL
CREATE OR REPLACE PROCEDURE add_order_line(p_order IN NUMBER, p_prod IN NUMBER,
                                           p_qty IN NUMBER, p_line OUT NUMBER) IS
BEGIN
  UPDATE product SET stock = stock - p_qty WHERE product_id = p_prod AND stock >= p_qty;
  IF SQL%ROWCOUNT = 0 THEN
    RAISE_APPLICATION_ERROR(-20001, 'Out of stock');
  END IF;
  INSERT INTO order_line(order_id, product_id, qty) VALUES (p_order, p_prod, p_qty)
  RETURNING line_id INTO p_line;
  -- transaction control is the caller's responsibility (COMMIT/ROLLBACK)
END;
```
```sql
-- PL/pgSQL
CREATE OR REPLACE PROCEDURE add_order_line(p_order int, p_prod int, p_qty int, INOUT p_line int DEFAULT NULL)
LANGUAGE plpgsql AS $$
BEGIN
  UPDATE product SET stock = stock - p_qty WHERE product_id = p_prod AND stock >= p_qty;
  IF NOT FOUND THEN
    RAISE EXCEPTION 'Out of stock' USING ERRCODE = 'P0001';
  END IF;
  INSERT INTO order_line(order_id, product_id, qty) VALUES (p_order, p_prod, p_qty)
  RETURNING line_id INTO p_line;
END;
$$;
```
Differences to note: T-SQL manages the transaction *inside* the procedure and cares about `XACT_ABORT`; PL/SQL typically leaves it to the caller; PL/pgSQL's exception block creates an implicit **savepoint** (and has a performance cost); the `CREATE OR ALTER` / `CREATE OR REPLACE` / `CREATE OR REPLACE` spellings differ even for DDL.

### 5. Dynamic SQL

```sql
-- T-SQL: ALWAYS parameterise values; QUOTENAME for identifiers
DECLARE @sql nvarchar(max) =
  N'SELECT * FROM ' + QUOTENAME(@schema) + N'.' + QUOTENAME(@table) + N' WHERE Id = @id;';
EXEC sys.sp_executesql @sql, N'@id int', @id = @id;
```
| Aspect | T-SQL | PL/SQL | PL/pgSQL |
|---|---|---|---|
| Execute | `sp_executesql`, `EXEC (@sql)` | `EXECUTE IMMEDIATE` | `EXECUTE` |
| Identifier quoting | `QUOTENAME()` | `DBMS_ASSERT.ENQUOTE_NAME` | `format('%I', name)` / `quote_ident` |
| Value binding | named parameters | `USING` | `USING` |
| Scope of caller's variables/temp tables | dynamic batch **can** see `#temp` of caller but **not** `@vars`/table variables | no local variable visibility | no local variable visibility |

Pitfall: `EXEC(@sql)` concatenated with user input is SQL injection; in ADF the equivalent mistake is building SQL with `@concat(...)` from pipeline parameters.

### 6. Functions: what can be done inside a query

| Capability | T-SQL scalar UDF | Inline TVF | Multi-statement TVF |
|---|---|---|---|
| Can be used in `SELECT`/`WHERE` | yes | yes (as table) | yes (as table) |
| Side effects (DML on permanent tables) | **not allowed** | not allowed | not allowed |
| Cost model | Historically per-row, serialising; **inlined** automatically in 2019+ when eligible | Expanded into the calling query — like a parameterised view | Opaque table with fixed-guess cardinality (1 or 100 rows; interleaved execution fixes some cases from 2017) |
| Prefer | Only when inlining applies | **Yes** | Rarely |

PL/SQL allows functions with side effects (restricted in SQL context); PostgreSQL has `IMMUTABLE/STABLE/VOLATILE` categories that guide the planner, which T-SQL lacks — T-SQL uses *schemabinding* and determinism checks instead.

### 7. Under the hood

- **Compilation unit:** T-SQL procedures are compiled **per statement**; recompilation happens on schema change, statistics change, `OPTION (RECOMPILE)` or `WITH RECOMPILE`. PL/SQL compiles whole units to p-code/native; PL/pgSQL is interpreted, with plan caching per statement per session.
- **Plan reuse:** Because the first execution parameters are sniffed, a procedure's behaviour may vary by its first caller — see [Parameter Sensitivity](<../../04_Query_Processing/Parameter_Sensitivity/_index.md>).
- **Context switches:** In PL/SQL each SQL call from PL/SQL pays a PL/SQL↔SQL engine switch; in T-SQL there is no such split, but each statement still goes through the executor's setup path.

### 8. Performance and design implications

- Replace cursors with set-based DML; if you must loop, loop over *batches* (`TOP (5000)` + `OUTPUT`), not rows.
- Use TVPs or `OPENJSON` to pass sets into procedures instead of comma-separated strings or loops.
- Keep procedure result shapes stable — tools (ADF, Power BI, ORMs) infer metadata from the first result set; conditional different `SELECT`s break them (`sys.dm_exec_describe_first_result_set` shows what a tool will see).
- Prefer `CREATE OR ALTER` (2016 SP1+) for idempotent deployments.

### 9. Common mistakes

- Using cursors for row-wise updates.
- Leaving `PRINT`/debug `SELECT` in procedures called by orchestration tools.
- Forgetting `SET NOCOUNT ON` → extra "n rows affected" messages confuse some drivers and slow chatty loops.
- Using `@@IDENTITY` (trigger-polluted) rather than `SCOPE_IDENTITY()` or `OUTPUT`.
- Transliterating PL/SQL `EXCEPTION WHEN NO_DATA_FOUND` — T-SQL has no exception for "no row"; check `@@ROWCOUNT` / `IF NOT EXISTS`.

### 10. 🎯 Interview kit

**30-second answer:** "T-SQL is both the query language and the procedural language of SQL Server; Oracle splits it as SQL + PL/SQL, PostgreSQL as SQL + PL/pgSQL. Big practical differences: batches and `GO`, implicit result sets from procedures, statement-level triggers with `inserted`/`deleted`, `TRY…CATCH` + `XACT_ABORT`, and only a `WHILE` loop."

**Likely follow-ups:** *Row- vs statement-level triggers?* → T-SQL fires once per statement; use `inserted` set. *Why avoid scalar UDFs?* → per-row execution, serial plans, until inlining (2019). *Difference between `THROW` and `RAISERROR`?* → THROW preserves original error and requires a preceding `;`, RAISERROR formats a new message.

### 11. Check yourself

- [ ] Port the order-line procedure to MySQL.
- [ ] Explain why `SELECT @@ROWCOUNT` after an `IF` returns 0.
- [ ] Why does the variable `@x` vanish after `GO`?
- [ ] List three reasons to prefer an inline TVF over a scalar UDF.

**Where this connects:** [Programmable Objects](<../../06_T-SQL_Programming/Programmable_Objects/_index.md>) · [Temp Tables vs Table Variables](<../Temp_Tables_vs_Table_Variables.md>) · [MERGE Pitfalls and Error Handling](<../MERGE_Pitfalls_and_Error_Handling.md>) · [Oracle PL/SQL](<../../../Oracle/PL-SQL/_index.md>) · Next → [T-SQL Across the Microsoft Family](<08_T-SQL_Across_the_Microsoft_Family.md>)
<!-- explainer:end -->
