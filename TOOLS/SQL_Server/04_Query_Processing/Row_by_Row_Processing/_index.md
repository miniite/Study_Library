---
id: "TOOLS/SQL_Server/04_Query_Processing/Row_by_Row_Processing"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Row-by-Row vs Set-based

**L4 Advanced** · [↑ Query Processing and Tuning](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** SQL Server is built to process sets. Cursors and WHILE loops that handle one row at a time ("RBAR": row by agonizing row) pay per-statement overhead on every row, so a set-based rewrite is usually 10× to 1,000× faster.

### 1. The big picture
Developers who come from procedural languages naturally think in loops: "for each customer, look up their orders, compute a total, update a row." T-SQL allows that with **cursors** and **WHILE loops**. But each iteration is a separate statement: it is parsed or plan-looked-up, executed, logged and locked on its own, and returns to the control-of-flow engine. A loop over 1 million rows costs about the same as running 1 million separate statements.

A **set-based** statement describes the whole result at once. The optimizer then chooses one efficient plan (scans, hash joins, batch mode, parallelism) that processes all rows in a single pass, with minimal logging overhead per row.

Modern T-SQL has set-based answers to nearly every classic cursor use: window functions (running totals, ranking, previous/next row), `PIVOT` or conditional aggregation, `MERGE`/`UPDATE … FROM`, `STRING_AGG`, recursive CTEs, `GENERATE_SERIES`, `APPLY`. Cursors still have legitimate uses, mainly **administrative** tasks that must call a procedure or run dynamic SQL once per object, or work that must be processed in small batches for concurrency reasons.

### 2. Core concepts and vocabulary
- **Set-based**: one statement operating on all qualifying rows.
- **Iterative / procedural**: code that loops with `WHILE`, fetching or modifying one row (or one small group) per iteration.
- **Cursor**: a server-side object that holds a result set and lets you `FETCH` rows one by one: `DECLARE … CURSOR`, `OPEN`, `FETCH NEXT … INTO`, `@@FETCH_STATUS`, `CLOSE`, `DEALLOCATE`.
- **`@@FETCH_STATUS`**: 0 = a row was fetched; −1 = no more rows (past the end); −2 = the row is missing (deleted in a keyset cursor). The loop runs **while it equals 0**.
- **Cursor types**: `STATIC` (copy in tempdb), `KEYSET` (keys in tempdb), `DYNAMIC` (sees changes), `FAST_FORWARD` (forward-only, read-only, optimized). Default cursors are global, scrollable-capable and updatable, which is heavier than needed.
- **`LOCAL FAST_FORWARD`**: the cheapest cursor option when a cursor really is required.
- **Batching**: processing a large modification in chunks (for example 5,000 rows per loop) to limit log growth and lock escalation. It is a loop, but each iteration is set-based. This is a good pattern.

### 3. How it works, step by step
**What one cursor iteration costs:**
1. `FETCH NEXT` reads the next row from the cursor's work table or the base table (worktable in tempdb for STATIC/KEYSET).
2. The loop body runs one or more statements. Each one needs plan lookup, execution context, latches and locks, and log records, with a commit per statement if no explicit transaction is open.
3. Control returns to the T-SQL interpreter, which evaluates `@@FETCH_STATUS` and loops.

Multiply by N rows. A set-based statement pays the fixed costs **once**, and its per-row cost is a few CPU instructions inside an operator.

**The rewrite process:**
1. Write down what the loop computes **for the whole set**, not per row ("for every category, total quantity in stock, laid out as columns").
2. Find the set-based building block: aggregation, join, window function, pivot, `UPDATE … FROM` join, `MERGE`.
3. Write it as one statement (or a few with temp tables).
4. Compare results (`EXCEPT` both ways) and performance (`STATISTICS IO, TIME`).

**Classic cursor tasks and their set-based replacements:**
| Cursor task | Set-based replacement |
|---|---|
| Running total | `SUM(x) OVER (ORDER BY d ROWS UNBOUNDED PRECEDING)` |
| Ranking / numbering | `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`, `NTILE()` |
| Compare with previous row | `LAG()` / `LEAD()` |
| Pivot rows to columns | `PIVOT`, or `SUM(CASE WHEN cat = 'X' THEN qty END)` |
| Update each row from another table | `UPDATE t SET … FROM t JOIN s ON …` |
| Insert-or-update | `MERGE`, or `UPDATE` then `INSERT … WHERE NOT EXISTS` |
| Concatenate values | `STRING_AGG()` |
| Walk a hierarchy | Recursive CTE |
| Generate rows/dates | `GENERATE_SERIES()` (2022) or a numbers/tally table |
| Call a function per row | `CROSS APPLY` (with an inline TVF) |

### 4. Worked example
**a) Loop vs set: inserting rows**
```sql
SET NOCOUNT ON;
DROP TABLE IF EXISTS dbo.demoPerformance;
CREATE TABLE dbo.demoPerformance (CustomerID int);

DECLARE @t0 datetime2 = SYSDATETIME();
INSERT INTO dbo.demoPerformance (CustomerID)
SELECT CustomerID FROM Sales.Customer;               -- set-based: ~20k rows
SELECT DATEDIFF(ms, @t0, SYSDATETIME()) AS set_ms;

TRUNCATE TABLE dbo.demoPerformance;
SET @t0 = SYSDATETIME();
WHILE EXISTS (SELECT * FROM Sales.Customer AS c
              WHERE NOT EXISTS (SELECT * FROM dbo.demoPerformance AS p
                                WHERE p.CustomerID = c.CustomerID))
BEGIN
  INSERT INTO dbo.demoPerformance (CustomerID)
  SELECT TOP (1) CustomerID FROM Sales.Customer AS c
  WHERE NOT EXISTS (SELECT * FROM dbo.demoPerformance AS p
                    WHERE p.CustomerID = c.CustomerID);
END
SELECT DATEDIFF(ms, @t0, SYSDATETIME()) AS loop_ms;  -- can take minutes
```
The set-based insert finishes in milliseconds. The loop runs two statements per row, and each `NOT EXISTS` re-scans a growing heap, so the cost grows roughly with N². Be ready to cancel it.

**b) Cursor pivot vs conditional aggregation**
A report needs stock quantity per product category as **columns**. The cursor version creates a temp table with one column per category, inserts a zero row, then loops over `(Category, Qty)` rows, running an `UPDATE` per row to add to the right column (with dynamic SQL or a long `IF` chain). The set-based version is one statement:
```sql
SELECT
  SUM(CASE WHEN pc.Name = 'Bikes'       THEN pi.Quantity ELSE 0 END) AS Bikes,
  SUM(CASE WHEN pc.Name = 'Components'  THEN pi.Quantity ELSE 0 END) AS Components,
  SUM(CASE WHEN pc.Name = 'Clothing'    THEN pi.Quantity ELSE 0 END) AS Clothing,
  SUM(CASE WHEN pc.Name = 'Accessories' THEN pi.Quantity ELSE 0 END) AS Accessories
FROM Production.ProductInventory AS pi
JOIN Production.Product AS p              ON p.ProductID = pi.ProductID
JOIN Production.ProductSubcategory AS ps  ON ps.ProductSubcategoryID = p.ProductSubcategoryID
JOIN Production.ProductCategory AS pc     ON pc.ProductCategoryID = ps.ProductCategoryID;
```
(`PIVOT` gives the same result. Conditional aggregation is often easier to extend.)

**c) When a loop is right: batching a huge delete**
```sql
DECLARE @rows int = 1;
WHILE @rows > 0
BEGIN
  DELETE TOP (5000) FROM dbo.AuditLog WHERE LoggedAt < '20230101';
  SET @rows = @@ROWCOUNT;
END
```
Each iteration is set-based (5,000 rows). The loop exists to keep transactions short, avoid lock escalation (which starts around 5,000 locks) and let the log be reused. This is the correct way to use a loop.

**d) If you must use a cursor (admin task)**
```sql
DECLARE @name sysname, @sql nvarchar(max);
DECLARE dbs CURSOR LOCAL FAST_FORWARD FOR
  SELECT name FROM sys.databases WHERE state_desc = N'ONLINE' AND database_id > 4;
OPEN dbs;
FETCH NEXT FROM dbs INTO @name;
WHILE @@FETCH_STATUS = 0          -- loop while a row was fetched
BEGIN
  SET @sql = N'USE ' + QUOTENAME(@name) + N'; EXEC sp_updatestats;';
  EXEC sys.sp_executesql @sql;
  FETCH NEXT FROM dbs INTO @name;
END
CLOSE dbs; DEALLOCATE dbs;
```
Note `LOCAL FAST_FORWARD`, `QUOTENAME` for safety, and `CLOSE`/`DEALLOCATE`.

### 5. Under the hood
- **Per-statement overhead**: every statement in a loop gets its own execution context, lock acquisition and release, and (in autocommit mode) its own transaction with a **log flush on commit**. Log flushes are synchronous disk writes, often the biggest cost of row-by-row DML. Wrapping a loop in one explicit transaction removes the per-row flushes (but holds locks longer). A set-based statement writes its log records in one transaction.
- **Plan quality**: the optimizer sees a single-row statement each time, so it can't choose hash joins, batch mode or parallelism, which only pay off on sets.
- **Cursor storage**: `STATIC` and `KEYSET` cursors materialize data or keys into **tempdb** worktables on `OPEN`. `DYNAMIC` cursors re-evaluate on each fetch. `FAST_FORWARD` lets the engine pick the cheapest implementation. The default global cursor scope can leak between batches in the same connection if not deallocated.
- **Hidden RBAR**: scalar UDFs called in a SELECT (when not inlined), multi-statement TVFs, triggers written for single rows, and correlated subqueries executed per row through nested loops all behave like row-by-row code, even when the outer statement looks set-based. Look for high *Number of Executions* in plans.
- **Window functions** use the Segment/Sequence Project/Window Spool operators (or the batch-mode Window Aggregate operator from 2016+), which compute running values in one ordered pass. That is why they beat cursor-based running totals by large margins.

### 6. Performance and design implications
- Treat any loop over data rows as a code-review flag. Ask "what is the set-based form?"
- Measure: set-based rewrites typically drop elapsed time from minutes to seconds, and reads by orders of magnitude.
- Loops are fine when each iteration processes a meaningful set (batching), or per-object admin work.
- Single-row-minded triggers (`SELECT @id = id FROM inserted`) are both slow and **wrong** for multi-row statements. Always write triggers against the whole `inserted`/`deleted` sets.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `DECLARE c CURSOR LOCAL FAST_FORWARD FOR …` | Cheapest cursor | Unavoidable per-row admin tasks |
| `@@FETCH_STATUS` | 0 = row fetched, −1 = end, −2 = missing | Loop condition |
| Window functions (`OVER (…)`) | Running totals, ranks, LAG/LEAD | Replacing ordered cursors |
| `UPDATE … FROM … JOIN` / `MERGE` | Set-based modifications from another table | Replacing per-row updates |
| `DELETE TOP (n)` + `@@ROWCOUNT` loop | Batched modification | Huge deletes/updates |
| `STRING_AGG`, `PIVOT`, `GENERATE_SERIES` | Set-based string, pivot and row generation | Replacing loops |
| `SET NOCOUNT ON` | Removes per-statement messages | Any loop or procedure |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Set-based statement: almost always | A single giant statement would hold locks or grow the log too much → batch it |
| Batching loop (set per iteration) | Batches so small they become RBAR again |
| `LOCAL FAST_FORWARD` cursor: per-object admin, calling procs per row | Processing business data |
| Explicit transaction around a necessary loop | Long transactions blocking others |

### 9. Common mistakes
- Using a cursor for running totals, pivots or ranking → slow → window functions, conditional aggregation.
- Writing the loop condition backwards (`WHILE @@FETCH_STATUS <> 0`) → loop never runs or never ends → loop while it equals 0.
- Default cursor options → heavier global, updatable cursors → `LOCAL FAST_FORWARD`.
- Forgetting `CLOSE`/`DEALLOCATE` → "cursor already exists" errors and held resources → always clean up (or use LOCAL).
- Triggers that assume one row → wrong results for multi-row statements → join to `inserted`.
- Scalar UDF in a WHERE on millions of rows → hidden row-by-row execution → inline it or rewrite as an inline TVF/expression.

### 10. 🎯 Interview kit
**30-second answer:** "Row-by-row code like cursors and WHILE loops runs a separate statement per row, paying plan lookup, locking, logging and commit costs each time, and prevents the optimizer from using set-oriented plans like hash joins, batch mode and parallelism. I rewrite them with set-based constructs: window functions for running totals and LAG/LEAD, conditional aggregation or PIVOT, UPDATE FROM or MERGE, STRING_AGG. Loops are fine for batching large modifications or per-object admin work, using LOCAL FAST_FORWARD cursors."
**Structure a longer answer:** why sets win (fixed costs once, better plans) → rewrite catalogue → legitimate loops (batching, admin) → hidden RBAR (scalar UDFs, single-row triggers) → measure.
**Likely follow-ups:** *Is a cursor ever acceptable?* → admin tasks, calling procedures per row; use LOCAL FAST_FORWARD. *How do you delete 100 million rows safely?* → batched DELETE TOP (n) loop. *What's hidden RBAR?* → scalar UDFs, nested-loop correlated subqueries, MSTVFs.
**Signals interviewers look for:** thinks in sets; knows window functions; knows when loops are right (batching); spots hidden row-by-row code.

### 11. Check yourself
- [ ] Time the loop vs set-based insert and explain why the loop grows faster than linearly.
- [ ] Replace a cursor-based running total with a window function and compare reads.
- [ ] Write a pivot report with conditional aggregation.
- [ ] Write a batched delete and explain why 5,000 is a common batch size.
- [ ] Rewrite a trigger that does `SELECT @id = id FROM inserted` to handle multiple rows.
<!-- explainer:end -->

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 16: Best Practices for T-SQL Development and Database Design › Cursors for Row-by-Row Processing
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.)
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 16: Best Practices for T-SQL Development and Database Design › Use Set-Based Logic, Avoid RBAR (Row-By-Agonizing-Row)
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 18 Row-By-Row Processing From Cursors And Other Causes
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Temporary Tables and Table Variables › Using a Cursor
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Overusing Cursors
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — T-SQL development › 5.7 #20 Using cursors
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Manipulating Data › Thinking About Performance
- *Practical Graph Structures in SQL Server and Azure SQL* — Louis Davidson — Tree Structures, Algorithms, and Performance › Helper Table
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Programmable objects

