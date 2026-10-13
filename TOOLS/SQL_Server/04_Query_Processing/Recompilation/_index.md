---
id: "TOOLS/SQL_Server/04_Query_Processing/Recompilation"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Recompilation

**L4 Advanced** · [↑ Query Processing and Tuning](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A recompile throws away a cached plan for one statement and optimizes it again, because something it depends on changed (schema, statistics, SET options, a new object) or because you asked for it. Recompiles keep plans correct and current, but too many of them burn CPU and stall callers.

### 1. The big picture
The plan cache lesson showed why reuse saves CPU. Reuse has a limit: a plan is built for one version of the world, meaning one schema, one set of statistics and certain SET options. When that world changes, the cached plan may be invalid (it uses an index that was dropped) or just out of date (data has grown 100×). **Recompilation** is SQL Server's way of keeping plans in step with reality.

Recompilation is therefore a balance:
- **Good:** a new index, a statistics update or a big change in data gives the optimizer better information, and a new plan can be much faster.
- **Bad:** every recompile costs CPU, and while a statement recompiles, other callers needing that plan wait. Code that triggers a recompile on every execution (DDL mixed with DML, changing SET options inside procedures) can make a busy system CPU-bound.

### 2. Core concepts and vocabulary
- **Compile vs recompile**: a *compile* happens when no plan exists. A *recompile* replaces an existing plan that was found to be invalid or stale.
- **Statement-level recompilation**: since SQL Server 2005, only the affected **statement** is recompiled, not the whole procedure. You may see more recompile events, but each one is cheaper and blocks less.
- **Schema stability check**: before execution, SQL Server checks whether the objects a plan uses have changed (schema version).
- **Optimality check**: SQL Server checks whether statistics used by the plan have been updated (or would be auto-updated) since compilation.
- **Deferred name resolution**: a procedure can be created even though a table it references does not exist yet. The statement is compiled when it first runs after the object exists, which counts as a recompile.
- **Recompilation threshold (RT) / modification counter**: the number of row changes after which a table's statistics are considered stale. For temp tables the threshold is low (starts at 6 rows), which causes frequent recompiles.
- **`sp_recompile`**: marks an object so all plans that reference it recompile on next use.
- **`WITH RECOMPILE` / `OPTION (RECOMPILE)`**: ask for a fresh plan on every execution (procedure level / statement level).
- **KEEP PLAN / KEEPFIXED PLAN**: hints that relax (KEEP PLAN) or remove (KEEPFIXED PLAN) statistics-based recompiles for a statement.

### 3. How it works, step by step
At execution time, for each statement in a cached plan:
1. **Schema check.** Has any referenced table, view or temp table changed (columns, indexes, constraints, defaults/bindings)? If so → recompile.
2. **SET options check.** Do the session's plan-affecting SET options differ from those in the plan's cache key? Different options normally lead to a *different cached plan*. Changing a SET option *inside* a procedure forces the following statements to recompile.
3. **Statistics check.** Have the statistics used by this plan been updated since it was compiled, or has the table passed its modification threshold so that auto-update will refresh them now? If so → update stats (if needed) → recompile.
4. **Deferred objects.** Is this the first run of a statement whose table did not exist at compile time (for example a temp table created earlier in the same procedure)? → compile now.
5. **Explicit request.** `OPTION (RECOMPILE)`, `WITH RECOMPILE`, `sp_recompile`, or a Query Store forced hint that requires it.
6. The new plan for the statement replaces the old one in the cache. **If Query Store is forcing a plan** for that query, the recompile still happens, but the forced plan is used instead of the newly optimized one (unless the forced plan is no longer valid, for example its index was dropped).

**Main causes of recompiles:**
| Cause | Example |
|---|---|
| Schema change | `ALTER TABLE`, `CREATE INDEX` on a referenced table, temp table re-created |
| Binding change | A default or rule bound to a column changes |
| Statistics update | Auto-update after enough modifications, or manual `UPDATE STATISTICS` |
| Deferred object resolution | `CREATE TABLE #t` then `SELECT … FROM #t` in the same procedure |
| SET option change | `SET ANSI_NULLS OFF` inside a procedure |
| Explicit | `sp_recompile`, `WITH RECOMPILE`, `OPTION (RECOMPILE)` |

### 4. Worked example
**a) A recompile that helps**
```sql
CREATE OR ALTER PROCEDURE dbo.WorkOrder
AS
SELECT wo.WorkOrderID, wo.ProductID, wo.StockedQty
FROM Production.WorkOrder AS wo
WHERE wo.StockedQty BETWEEN 500 AND 700;
GO
EXEC dbo.WorkOrder;          -- plan: clustered index scan (no useful index)

CREATE INDEX IX_WorkOrder_StockedQty
  ON Production.WorkOrder (StockedQty) INCLUDE (ProductID);
EXEC dbo.WorkOrder;          -- schema changed → statement recompiles → index seek
```
The index changed the table's schema version, so the cached plan was checked, found stale, and recompiled. The second execution uses the new index, which is exactly what you want. (Clean up: `DROP INDEX IX_WorkOrder_StockedQty ON Production.WorkOrder;`)

**b) A recompile that costs: DDL mixed with DML**
```sql
CREATE OR ALTER PROCEDURE dbo.TempTableDemo
AS
CREATE TABLE #t (ID int);
INSERT INTO #t (ID) VALUES (1);       -- compiled at run time (deferred)
SELECT ID FROM #t;                    -- compiled at run time (deferred)
CREATE INDEX ix_id ON #t (ID);        -- DDL in the middle...
SELECT ID FROM #t WHERE ID = 1;       -- ...schema changed → recompile
GO
```
Put all DDL (including index creation on the temp table) at the **top**, or create the index inline (`CREATE TABLE #t (ID int INDEX ix_id)`), so later statements don't recompile on every call.

**c) Watching recompiles with Extended Events**
```sql
CREATE EVENT SESSION QueryRecompile ON SERVER
ADD EVENT sqlserver.sql_statement_recompile
  (ACTION (sqlserver.sql_text, sqlserver.database_name)
   WHERE (sqlserver.database_name = N'AdventureWorks'))
ADD TARGET package0.event_file (SET filename = N'QueryRecompile');
ALTER EVENT SESSION QueryRecompile ON SERVER STATE = START;
```
The `recompile_cause` field tells you *why* (Schema changed, Statistics changed, Deferred compile, Set option change, Option (recompile) requested…). To see exactly which statement inside a procedure recompiled, also capture `sp_statement_starting/completed` and turn on **causality tracking** (`TRACK_CAUSALITY = ON`), then order events by `attach_activity_id`.

Quick counters: perfmon *SQLServer:SQL Statistics → SQL Re-Compilations/sec* compared with *Batch Requests/sec*, or `plan_generation_num` in `sys.dm_exec_query_stats` (it goes up with each recompile of that statement).

### 5. Under the hood
- **Why temp tables recompile more than table variables:** temp tables have statistics, so they take part in optimality checks with a small threshold (6 rows, then 500, then 500 + 20% of rows, or the dynamic threshold under compatibility level 130+). Table variables have no statistics, so data changes in them never trigger statistics recompiles. The flip side is poor estimates: before SQL Server 2019 a table variable was estimated at 1 row. Since 2019 (compatibility level 150) *table variable deferred compilation* uses the actual row count at first compile.
- **Temp table caching:** a temp table created inside a procedure, with no DDL after creation and no named constraints, can be **cached** and reused between calls. Its statistics may then be stale from a previous call. Adding DDL after creation (like `CREATE INDEX` in the middle) prevents caching.
- **Compile locks:** while a statement compiles, other sessions that need the same plan wait (on a compile lock or wait types such as `RESOURCE_SEMAPHORE_QUERY_COMPILE` when many compiles need memory at once).
- **`OPTION (RECOMPILE)` optimizations:** because the plan is never reused, the optimizer can treat parameter and variable values as **constants** (*parameter embedding optimization*). That allows branch elimination for patterns like `WHERE (@x IS NULL OR col = @x)`.

### 6. Performance and design implications
- Some recompiles are healthy. Investigate when *Re-Compilations/sec* is a large fraction of *Batch Requests/sec* (more than roughly 10%) or compile CPU is high.
- Avoid recompile triggers in hot code: DDL after DML, SET option changes inside procedures, temp tables created outside the procedure that uses them, and unqualified object names.
- Prefer table variables (or temp tables with `KEEPFIXED PLAN`) for **small** working sets in very hot procedures. Prefer temp tables when row counts are large or vary, because their statistics give better plans.
- Use `OPTION (RECOMPILE)` on purpose: it is a good fix for rarely run, parameter-sensitive or "catch-all" queries, and a bad idea for queries running hundreds of times per second.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `OPTION (RECOMPILE)` | Fresh plan for this statement every run; embeds parameter values | Rare queries with very different parameter values |
| `CREATE PROCEDURE … WITH RECOMPILE` | Whole procedure compiled every run, never cached | Rarely; prefer the statement-level hint |
| `EXEC proc WITH RECOMPILE` | One-off fresh plan for this call | Testing |
| `EXEC sp_recompile N'dbo.Table'` | Marks plans referencing the object for recompile | After a big data change, to drop a bad plan |
| `OPTION (KEEP PLAN)` | Temp table uses normal (permanent table) threshold | Temp-table-heavy procedures |
| `OPTION (KEEPFIXED PLAN)` | No statistics-based recompiles | Plan is stable regardless of data |
| XE `sql_statement_recompile` | Event with `recompile_cause` | Finding why statements recompile |
| `sys.dm_exec_query_stats.plan_generation_num` | Number of times a statement's plan was regenerated | Quick spotting |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Let recompiles happen after index/stats changes | Normal case; this is how plans stay good |
| `OPTION (RECOMPILE)`: rare, skewed or catch-all queries | High-frequency OLTP (compile CPU dominates) |
| Table variable: tiny row counts, very hot code | Large or varying row counts (poor estimates before 2019) |
| Temp table: larger sets, need statistics/indexes | Interleaving DDL and DML, which causes recompiles |
| `KEEPFIXED PLAN`: stable plan, volatile data | Data distribution truly changes plan choice |

### 9. Common mistakes
- Creating indexes on temp tables halfway through a procedure → recompile every call, no temp table caching → declare indexes inline at creation.
- `SET ANSI_NULLS`/`ARITHABORT` changes inside procedures → recompiles and plan duplicates → set options consistently at the connection level.
- Putting `WITH RECOMPILE` on a whole procedure to fix one statement → every statement compiles each time → `OPTION (RECOMPILE)` on the one statement.
- Using `OPTION (RECOMPILE)` on a query called 5,000 times/second → CPU saturation → use a stable plan (Query Store hint, OPTIMIZE FOR, or code split) instead.
- Blaming "recompiles" without checking the cause → wrong fix → read `recompile_cause` in Extended Events.

### 10. 🎯 Interview kit
**30-second answer:** "SQL Server recompiles a statement when its cached plan may be invalid or stale: schema changes, statistics updates, deferred objects like temp tables, SET option changes, or an explicit RECOMPILE. Since 2005 it's statement-level. Recompiles keep plans current but cost CPU and block callers, so I check the cause with the sql_statement_recompile Extended Event and fix code patterns like DDL interleaved with DML, while using OPTION (RECOMPILE) deliberately for rare, parameter-sensitive queries."
**Structure a longer answer:** why plans become stale → what triggers recompiles → how to detect (XE, perfmon, plan_generation_num) → code patterns to avoid → when to ask for recompiles.
**Likely follow-ups:** *Temp table vs table variable for recompiles?* → temp tables have stats and recompile; table variables don't (but estimate poorly before 2019). *What does OPTION (RECOMPILE) do beyond recompiling?* → embeds parameter values as constants. *How does Query Store forcing interact with recompiles?* → recompile happens, forced plan is used.
**Signals interviewers look for:** sees recompiles as both helpful and costly; knows causes; uses Extended Events; knows statement-level recompilation.

### 11. Check yourself
- [ ] Show a procedure picking up a new index after a schema-change recompile.
- [ ] Capture `sql_statement_recompile` and read `recompile_cause` for a temp table procedure.
- [ ] Rewrite a procedure so its temp table DDL no longer causes recompiles.
- [ ] Explain why temp tables cause more statistics-based recompiles than permanent tables.
- [ ] Explain when `OPTION (RECOMPILE)` is a good fix and when it is harmful.
<!-- explainer:end -->

## References

- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 8 Query Recompilation
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.)
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Monitoring Compilation and Recompilation Events
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Error handling, testing, source control, and deployment › 7.4 #29 Not making use of Schema Compare
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 15 Reduce Query Resource Use
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Impact Analysis › Save Index Changes to Source Code Repository
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 16: Best Practices for T-SQL Development and Database Design › Schema Versioning and Migrations
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Temporary Tables and Table Variables › Creating Global Temp Tables
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Programmable objects
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Managing Transactions › Rolling Back a Transaction

