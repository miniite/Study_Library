---
id: "TOOLS/SQL_Server/T-SQL/Temp_Tables_vs_Table_Variables"
level: 4
type: "concept"
topic: "TOOLS/SQL_Server/T-SQL"
tools: ["SQL_Server"]
---

# Temp Tables vs Table Variables

**L4 Advanced** · [↑ T-SQL](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A `#temp` table is a real tempdb table with statistics, indexes and normal transactional behaviour, while a `@table variable` is a lighter-weight tempdb object with no column statistics (and a fixed "1-row" guess before SQL Server 2019) — so use temp tables for anything more than a few hundred rows or anything the optimizer must reason about.

### 1. The big picture
T-SQL code often needs somewhere to hold an intermediate result: staging rows between steps of a procedure, breaking a monster query into pieces, passing data between procedures, or capturing OUTPUT rows. SQL Server offers temporary tables (`#t`, `##t`), table variables (`@t`), table-valued parameters, CTEs (not materialised storage — just a named query) and derived tables. The choice between `#temp` and `@table` is about **statistics, recompilation, scope and transaction semantics**, not about "memory vs disk" — both live in tempdb (and can be spilled to disk or stay in the buffer pool).

### 2. Core concepts and vocabulary
- **Local temporary table `#t`** — visible to the creating session (and nested procedures it calls); dropped automatically at the end of the session or the creating procedure.
- **Global temporary table `##t`** — visible to all sessions; dropped when the creating session ends *and* no other session is still using it.
- **Table variable `@t`** — a variable of type table, scoped to the batch/procedure/function in which it is declared.
- **Table-valued parameter (TVP)** — a table-typed parameter (user-defined table type) passed read-only to a procedure.
- **Statistics** — histograms/density info the optimizer uses to estimate cardinality.
- **Recompilation** — a statement's plan is rebuilt, typically because statistics changed enough.
- **tempdb** — the system database holding temp objects, worktables, row versions and spills.
- **Temp table caching** — SQL Server reuses a cached temp-table structure across procedure executions to cut allocation cost.
- **Deferred compilation** — SQL Server 2019+ (compat level 150): statements that reference a table variable are compiled at first execution when its row count is known.

### 3. How it works, step by step
1. `CREATE TABLE #t (…)` (or `SELECT … INTO #t`) allocates a table in tempdb under a name suffixed with a unique session id.
2. Rows inserted are logged in tempdb (minimal logging is possible with `SELECT INTO`, and tempdb logging is lighter: no redo needed after a restart).
3. As rows change, the **auto-update statistics** thresholds are tracked; when crossed, queries using the table recompile and see accurate estimates.
4. When a procedure ends, the temp table is dropped (or its structure is returned to the cache).
5. A table variable is created in the same way, but no column statistics are maintained; rollbacks do not undo its contents (it ignores the user transaction), and it never causes statistics-based recompiles.

| Property | `#temp` | `@table` |
|---|---|---|
| Column statistics | Yes (auto-created, auto-updated) | No (only index/PK stats are not shared; estimates use row count at best) |
| Cardinality estimate | Based on stats; recompile when data changes | Pre-2019: 1 row (when compiled); 2019+: actual row count at first compile, but still no value distribution |
| Indexes after creation | `CREATE INDEX` any time | Only inline in the declaration (and from 2014, inline non-unique indexes) |
| Transaction behaviour | Fully transactional (rolled back with the transaction) | Not affected by rollback of the outer transaction |
| Scope | Session/procedure and nested calls | The declaring batch/module only |
| Parallel plans | Allowed | `INSERT…SELECT` into a table variable cannot go parallel (reads can) |
| Schema changes | `ALTER TABLE` allowed | Not allowed |
| Can be passed to procedure | Visible to called procedures automatically | Only as a read-only TVP |
| Recompiles | Can trigger (stats / schema) | Doesn't trigger stats recompiles |

### 4. Worked example
```sql
-- Large intermediate result: use a temp table
DROP TABLE IF EXISTS #big;
SELECT o.CustomerID, o.OrderID, o.OrderDate, SUM(d.Quantity * d.UnitPrice) AS Total
INTO   #big
FROM   Sales.Orders o
JOIN   Sales.OrderDetails d ON d.OrderID = o.OrderID
WHERE  o.OrderDate >= '2025-01-01'
GROUP  BY o.CustomerID, o.OrderID, o.OrderDate;

CREATE CLUSTERED INDEX cx_big ON #big (CustomerID, OrderDate);

SELECT c.CustomerName, b.Total
FROM   #big b
JOIN   Sales.Customers c ON c.CustomerID = b.CustomerID
WHERE  b.Total > 10000;
```
```sql
-- Small, bounded set, no need for stats: table variable is fine
DECLARE @ids TABLE (OrderID int PRIMARY KEY);
INSERT @ids VALUES (10248),(10249),(10250);

SELECT * FROM Sales.Orders WHERE OrderID IN (SELECT OrderID FROM @ids);
```
```sql
-- Demonstrating rollback behaviour
CREATE TABLE #t (i int);
DECLARE @v TABLE (i int);
BEGIN TRAN;
  INSERT #t VALUES (1);
  INSERT @v VALUES (1);
ROLLBACK;
SELECT COUNT(*) FROM #t;   -- 0
SELECT COUNT(*) FROM @v;   -- 1  (table variables are not rolled back)
```
Reading the plan: with a large `@table`, the actual-vs-estimated row count on the scan of the variable shows `Estimated = 1`, `Actual = 200000` (pre-2019), and the plan picks nested loops with an index seek per row. The same query against a `#temp` with a good estimate chooses a hash join. Compare with *Include Actual Execution Plan* or `SET STATISTICS IO, TIME ON`.

### 5. Under the hood
- **Why estimates matter.** The optimizer decides join algorithm, join order, memory grant and parallelism from cardinality. A 1-row guess leads to nested loops with huge repeated seeks, tiny memory grants (leading to spills), and serial plans. Temp tables get statistics, so the plan adapts — at the price of extra recompilations.
- **Table variable improvements.** SQL Server 2012 added trace flag 2453 (recompile when row count changes); `OPTION (RECOMPILE)` on a statement lets it see the real row count; 2019 introduced *table variable deferred compilation*, which fixes the cardinality guess (but not the distribution of values).
- **Recompile cost.** Statistics-driven recompiles on temp tables can become a cost in high-frequency procedures; mitigate with `KEEP PLAN` / `KEEPFIXED PLAN` hints (trade accuracy for stability), or choose `@table` for tiny sets.
- **Temp table caching.** If a procedure creates a `#t` without named constraints/explicit DDL after creation, the object can be reused, avoiding allocation overhead and tempdb metadata contention. Altering the table or creating named constraints after creation disables caching.
- **tempdb contention.** Many sessions creating temp objects hit allocation pages (PFS, GAM, SGAM). Mitigation: multiple equal-sized tempdb data files (up to 8 per CPU group rule of thumb), memory-optimized tempdb metadata in SQL Server 2019+, and avoiding needless creation/drop cycles.
- **Memory-optimized table variables** (In-Memory OLTP) live in memory and avoid tempdb entirely, useful for very hot, small, short-lived sets, but they have limits.
- **Logging.** Table variable and temp table modifications are logged in tempdb's transaction log (needed for statement rollback), but tempdb never needs redo.

### 6. Performance and design implications
- **Rule of thumb**: > ~100–1,000 rows, joins to big tables, or need indexes/statistics → `#temp`. Few rows, one-pass, inside a function or needing to survive a rollback → `@table`. Functions can't use `#temp`.
- Create indexes *after* bulk-loading the temp table (or define a PK inline when it dictates the join order).
- Avoid `SELECT *` into temp tables; carry only needed columns, because every column occupies tempdb space and I/O.
- Monitor tempdb: `sys.dm_db_file_space_usage`, `sys.dm_db_session_space_usage`, waits `PAGELATCH_*` on 2:1:1/2:1:2/2:1:3, version store, and spill warnings in plans.
- Prefer set-based pipelines; don't loop and insert row by row into temp objects.
- Compare alternatives: CTE (no materialisation; may be re-evaluated), derived table (inlined), indexed view (persisted), TVP (read-only input), `OPTION (RECOMPILE)` to get better plans.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE TABLE #t` / `SELECT … INTO #t` | Create local temp table | Intermediate results |
| `DROP TABLE IF EXISTS #t` | Safe re-run drop (2016+) | Script re-runs |
| `DECLARE @t TABLE (…)` | Table variable | Small sets, rollback-insensitive logging |
| `OPTION (RECOMPILE)` | Fresh plan with real row counts | Table variable with varied sizes |
| Compat level 150+ | Table variable deferred compilation | Fix estimates without code change |
| `KEEP PLAN` | Reduce recompiles from stats thresholds | Hot procedures |
| Trace flag 2453 | Recompile on table variable row-count change | Legacy versions |
| `sys.dm_db_session_space_usage` | tempdb usage by session | Find heavy users |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `#temp` for large or filtered/joined sets | Procedure runs thousands of times per second with tiny sets (recompile/allocation overhead) |
| `@table` for tiny sets and function bodies | The set can be big or its distribution matters |
| `SELECT INTO #t` for speed | You need exact data types/constraints (create explicitly) |
| TVP to pass rows into a procedure | You need to modify the passed rows (it's read-only) |
| Memory-optimized table variable | Large sets or features the engine doesn't support |

### 9. Common mistakes
- "Table variables live in memory, so they're faster" → myth; both use tempdb → measure.
- Large `@table` joined to big tables → 1-row estimate, nested loops disaster → use `#temp` or `OPTION (RECOMPILE)`.
- Creating indexes before the load → slower inserts → load first, then index.
- Reusing `##global` temp tables in app code → name collisions and concurrency bugs → use `#local` or real staging tables with a run id.
- Expecting `@table` rollback → data remains → design explicitly.
- Altering the temp table in the procedure → defeats caching and causes recompiles → define it fully at creation.

### 10. 🎯 Interview kit
**30-second answer:** Both are tempdb objects. A temp table has statistics, supports indexes and DDL after creation, and participates in transactions; a table variable has no column statistics (a 1-row estimate before 2019), is not rolled back and is scoped to the batch. Use temp tables for big or complex intermediate sets and table variables for tiny sets or functions.

**Structure a longer answer:** definition → statistics and plan consequences → transaction/scope differences → internals (caching, recompile, tempdb) → recommendation and measurement.

**Likely follow-ups:**
- *Local vs global temp tables?* → session vs all sessions; lifetime rules.
- *Why does a large table variable perform poorly?* → fixed estimate, no stats, serial insert.
- *What did SQL Server 2019 change?* → deferred compilation gives correct row counts at first compile.
- *How does temp table caching work?* → structure reused across executions when created without subsequent DDL/named constraints.

**Signals interviewers look for:** you cite statistics and cardinality estimates rather than "memory vs disk"; you know the 2019 change; you can read a plan for estimate mismatch.

### 11. Check yourself
- [ ] Demonstrate the rollback difference with a short script.
- [ ] Explain why a 200,000-row table variable makes a bad plan pre-2019.
- [ ] Choose between `#temp`, `@table`, TVP and CTE for four scenarios.
- [ ] Name three ways to reduce tempdb allocation contention.
- [ ] List what disables temp-table caching.

<!-- related-deep-dives -->

### 🔗 Related deep dives
- [Syntax Translation Cookbook (temp tables and variables)](<Dialects_and_Portability/03_Syntax_Translation_Cookbook.md>)
- [Procedural Code: T-SQL vs PL/SQL vs PL/pgSQL](<Dialects_and_Portability/07_Procedural_Code_T-SQL_vs_PLSQL_vs_PLpgSQL.md>)
- [T-SQL Across the Microsoft Family (temp tables on Synapse/Fabric)](<Dialects_and_Portability/08_T-SQL_Across_the_Microsoft_Family.md>)
- [Series hub: T-SQL, ANSI SQL and Dialects](<Dialects_and_Portability/_index.md>)
<!-- explainer:end -->

## Implementations (tools)

- [SQL Server](<../_index.md>)

## Practice questions

- [Basic] [What are the GETDATE SYSDATETIME and DATEADD functions used for?](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42893_What_are_the_GETDATE_SYSDATETIME_and_DATEADD_functions_used_.md>)
- [Intermediate] [What is the difference between a local temporary table (#temp) and a global temporary table (##temp)?](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42907_What_is_the_difference_between_a_local_temporary_table_temp_.md>)
- [Deep] [What is the difference between a temporary table and a table variable in T-SQL?](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42908_What_is_the_difference_between_a_temporary_table_and_a_table.md>)
- [Deep] [Explain how a computed column can be PERSISTED and indexed, and give a case where it improved query performance.](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42840_Explain_how_a_computed_column_can_be_PERSISTED_and_indexed_a.md>)
- [Deep] [Explain how you version-control database schema and deploy migrations in a CI/CD pipeline for SQL Server.](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42859_Explain_how_you_version-control_database_schema_and_deploy_m.md>)
- [Expert] [Describe an expand-contract migration to rename a column without breaking a live application reading and writing the table.](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42797_Describe_an_expand-contract_migration_to_rename_a_column_wit.md>)
- [Expert] [What are In-Memory OLTP tables and natively compiled procedures, and what workloads justify them despite their restrictions?](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42834_What_are_In-Memory_OLTP_tables_and_natively_compiled_procedu.md>)
- [Expert] [Explain how Change Tracking differs from Change Data Capture and when you would pick each for downstream sync.](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42843_Explain_how_Change_Tracking_differs_from_Change_Data_Capture.md>)

## Related concepts

- [Date and Time Types](<../SQL_Server_Platform/Date_and_Time_Types.md>) *(SQL Server Platform)*
- [Schema Evolution in Lakehouse Tables](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Schema_Evolution_in_Lakehouse_Tables.md>) *(SQL Fundamentals)*
- [Schema Migration Risks](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Schema_Migration_Risks.md>) *(DBMS Concepts)*

## References

- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Error handling, testing, source control, and deployment › 7.4 #29 Not making use of Schema Compare
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › Best Practices for Maintainable T-SQL
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Persisted vs. Non-Persisted Computed Columns
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Azure SQL Database Fundamentals
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 15: Case Studies and the Future of T-SQL › Temporal Tables and System-Versioned Data
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Temporary Tables and Table Variables
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Temporary Tables and Table Variables › Creating Local Temp Tables
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Temporary Tables and Table Variables › Creating Table Variables
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Temporary Tables and Table Variables › Using a Temp Table or Table Variable
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Temporal tables
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 19 Memory-Optimized Oltp Tables And Procedures
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Temporary Objects
- *SQL Server Tutorial* (YouTube series) — Part 34: Temporary tables in SQL Server ([watch](https://www.youtube.com/watch?v=oGuS1rdfaMI))
- *SQL Server Tutorial* (YouTube series) — Part 149: Temp tables in dynamic sql ([watch](https://www.youtube.com/watch?v=ZQvItdzB8to))

