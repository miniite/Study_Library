---
id: "TOOLS/SQL_Server/03_Statistics_and_Cardinality"
level: 3
type: "curriculum"
curriculum: "sql_server"
---

# Statistics and Cardinality

**L3 Practitioner** · [↑ SQL Server](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Statistics are small summaries of how values are distributed in a column; the optimizer uses them to guess how many rows each step of a query will produce (cardinality), and those guesses decide which plan you get.

### 1. The big picture
SQL Server uses a **cost-based optimizer**. For every query it considers several possible plans (seek or scan, nested loops or hash join, which table first) and picks the one with the lowest estimated cost. The cost of every operator depends mostly on one number: *how many rows will flow through it*. That number is the **cardinality estimate**. The optimizer cannot count rows at compile time; that would cost as much as running the query. So it reads **statistics** instead.

If statistics are accurate, estimates are close to reality and plans are good. If statistics are missing, stale or misleading, estimates go wrong. A plan built for 10 rows that meets 1,000,000 rows will be slow: nested loops that run a million times, memory grants that are too small and spill to tempdb, or serial plans where a parallel one was needed. The reverse also hurts: a plan built for a million rows that meets 10 wastes memory and scans whole tables.

This block of lessons covers:
- **Statistics Objects**: what a statistics object contains and how to inspect it.
- **Histogram and Density**: the exact math the optimizer uses to turn statistics into row estimates.
- **Cardinality Estimator**: the model (legacy vs modern) that combines estimates when there are several predicates and joins.
- **Statistics Maintenance**: how statistics are created and refreshed, automatically and by hand.

It connects backward to indexing (every index has a statistics object) and forward to query processing (plans, parameter sensitivity, plan cache), which all depend on these estimates.

### 2. Core concepts and vocabulary
- **Statistics object**: a stored summary of the values in one or more columns of a table or indexed view. It has three parts: header, density vector and histogram.
- **Histogram**: up to 200 *steps*, each describing a range of values of the **first** column: how many rows fall inside the range and how many equal its upper bound.
- **Density**: `1 / number of distinct values`. Low density = very selective (many distinct values). Used when the optimizer does not know the exact value.
- **Cardinality**: the number of rows. *Cardinality estimation* = predicting row counts for each operator.
- **Selectivity**: the fraction of rows a predicate keeps (0 to 1). Estimated rows = table rows × selectivity.
- **Cardinality estimator (CE)**: the component that turns statistics and predicates into estimates. There is a *legacy* model (version 70) and a *modern* model (version 120 and later).
- **Index statistics**: created automatically with every index; same name as the index.
- **Column statistics**: created automatically on single columns used in predicates (`_WA_Sys_...` names) or by `CREATE STATISTICS`.
- **Stale statistics**: statistics that no longer describe the data because many rows changed since the last update.

### 3. How it works, step by step
1. **A query is compiled.** The optimizer parses it and finds the columns used in `WHERE`, `JOIN`, `GROUP BY` and `ORDER BY`.
2. **It loads statistics** for those columns. If a column has none and `AUTO_CREATE_STATISTICS` is on, it builds a single-column statistic on the spot.
3. **It checks freshness.** If enough rows have changed since the statistic was last updated and `AUTO_UPDATE_STATISTICS` is on, it refreshes the statistic (synchronously by default) before going on.
4. **It estimates each predicate.** For `WHERE Color = 'Red'` it looks up 'Red' in the histogram. For an unknown value (local variable, `OPTIMIZE FOR UNKNOWN`) it uses the density.
5. **The CE combines estimates** for multiple predicates and joins using its model assumptions.
6. **Costs are computed** for each candidate plan from these row counts, and the cheapest plan is chosen and cached.
7. **At run time** the plan runs as compiled, even if the real row counts are very different. The gap between *estimated* and *actual* rows is the main clue that statistics or the CE misled the optimizer.

| Where statistics come from | Name looks like | Columns |
|---|---|---|
| Creating an index | same as the index | all key columns (histogram on first) |
| Auto-create on a predicate column | `_WA_Sys_00000003_1A2B3C4D` | one column |
| `CREATE STATISTICS` | your name | one or more, optional filter |

### 4. Worked example
```sql
-- Does the database create statistics automatically?
SELECT name, is_auto_create_stats_on, is_auto_update_stats_on,
       is_auto_update_stats_async_on
FROM sys.databases WHERE name = DB_NAME();

-- Two tables with opposite distributions, no indexes on the filter/join column
CREATE TABLE dbo.Test1 (Test1_C1 INT IDENTITY, Test1_C2 INT);
CREATE TABLE dbo.Test2 (Test2_C1 INT IDENTITY, Test2_C2 INT);
INSERT dbo.Test1 (Test1_C2) VALUES (1);
INSERT dbo.Test1 (Test1_C2) SELECT TOP (10000) 2 FROM sys.all_columns a CROSS JOIN sys.all_columns b;
INSERT dbo.Test2 (Test2_C2) VALUES (2);
INSERT dbo.Test2 (Test2_C2) SELECT TOP (10000) 1 FROM sys.all_columns a CROSS JOIN sys.all_columns b;

-- Run with "Include Actual Execution Plan"
SELECT t1.Test1_C2, t2.Test2_C2
FROM dbo.Test1 t1 JOIN dbo.Test2 t2 ON t1.Test1_C2 = t2.Test2_C2
WHERE t1.Test1_C2 = 2;

-- Which statistics did the query cause to be created?
SELECT OBJECT_NAME(s.object_id) AS table_name, s.name, s.auto_created, s.user_created
FROM sys.stats s WHERE s.object_id IN (OBJECT_ID('dbo.Test1'), OBJECT_ID('dbo.Test2'));
```
**What to look at:**
- In the actual plan, each operator shows *Estimated Number of Rows* and *Actual Number of Rows*. Here they match: about 10,000 from `Test1` and 1 from `Test2`. The optimizer knew the distribution even without indexes.
- `sys.stats` now lists `_WA_Sys_...` statistics with `auto_created = 1` on `Test1_C2` and `Test2_C2`. They were built during compilation.
- If you disable auto-create and repeat, the plan shows a *Columns With No Statistics* warning and the estimates become guesses.

### 6. Performance and design implications
- **Estimated vs actual rows is the first thing to check** in a slow plan. Off by 10× or more in either direction usually means a statistics or CE problem (or a non-SARGable predicate, or parameter sniffing).
- **Underestimates** cause small memory grants (spills to tempdb, warning triangles on Sort/Hash), and loop joins over huge inputs.
- **Overestimates** cause excessive memory grants (other queries wait on `RESOURCE_SEMAPHORE`), unnecessary parallelism, and scans where a seek was better.
- Statistics are cheap to store (a few KB each), but updating them costs reads. Keep auto-create and auto-update on, then add targeted manual maintenance.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `DBCC SHOW_STATISTICS (table, stat)` | Shows header, density vector and histogram | Understand an estimate |
| `sys.stats`, `sys.stats_columns` | Lists all statistics and their columns | Find auto/user stats |
| `sys.dm_db_stats_properties` | Last update, rows sampled, modifications since update | Check staleness |
| `ALTER DATABASE ... SET AUTO_CREATE_STATISTICS ON` | Auto-builds column stats | Leave on |
| `ALTER DATABASE ... SET AUTO_UPDATE_STATISTICS ON` | Auto-refreshes stale stats | Leave on |
| `UPDATE STATISTICS t [stat] WITH FULLSCAN` | Rebuilds stats from all rows | Skewed or critical tables |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Default auto stats: almost every database | Turning them off "for performance": estimates degrade silently |
| FULLSCAN updates for skewed, critical tables | Huge tables during business hours |
| Async auto-update for high-throughput OLTP | Workloads where the first query after a big load must get the new plan |

### 9. Common mistakes
- Disabling auto-update to avoid compile pauses → stale statistics, bad plans → enable async update instead.
- Assuming a nightly index rebuild refreshes *all* statistics → column statistics are not touched → update them separately.
- Reading only the estimated plan → you cannot see the misestimate → capture the actual plan or Query Store runtime stats.
- Blaming the optimizer for a bad plan without checking estimates → wrong fix (hints) → first compare estimated vs actual rows.

### 10. 🎯 Interview kit
**30-second answer:** Statistics describe how values are distributed in columns: a header, a density vector and a histogram of up to 200 steps on the first column. The cost-based optimizer uses them to estimate row counts per operator, and those estimates drive join type, access method, memory grant and parallelism. SQL Server creates them for indexes and, by default, for predicate columns, and refreshes them when enough rows change.

**Structure a longer answer:** clarify (cost-based optimization) → what's in a statistic → how estimates drive plan choices → how stats get stale and are maintained → a real case where estimated 1 row vs actual 1M caused a spill.

**Likely follow-ups:**
- *How do you know statistics are the problem?* → estimated vs actual rows; `sys.dm_db_stats_properties.modification_counter`.
- *Why only 200 steps?* → keeps stats small and fast to load; skewed values between steps get averaged.
- *Are statistics created on non-indexed columns?* → yes, auto-created single-column stats.

**Signals interviewers look for:** links statistics → estimates → plan choice; knows estimated vs actual comparison; doesn't recommend turning auto stats off.

### 11. Check yourself
- [ ] Explain why an optimizer needs row estimates before it runs a query.
- [ ] Name the three parts of a statistics object.
- [ ] Run a query on an unindexed column and find the `_WA_Sys_` statistic it created.
- [ ] In an actual plan, find an operator where estimated and actual rows differ by 10× and name two possible causes.
<!-- explainer:end -->

## In this section (study in order)

1. [Statistics Objects](<Statistics_Objects/_index.md>) (L4 Advanced)
2. [Statistics Maintenance](<Statistics_Maintenance/_index.md>) (L4 Advanced)
3. [Cardinality Estimator](<Cardinality_Estimator/_index.md>) (L4 Advanced)

## References

- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 5 Statistics, Data Distribution, And Cardinality
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Histogram
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Statistics

