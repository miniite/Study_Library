---
id: "01_Databases_and_SQL/Query_Performance/Execution_Plans/Diagnosing_Query_Regressions"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Query_Performance/Execution_Plans"
tools: []
---

# Diagnosing Query Regressions

**L4 Advanced** · [↑ Execution Plans](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A query regression is a query that used to be fast and is now slow; you diagnose it by proving it with a baseline, comparing the old and new plan, and identifying which of a short list of causes — statistics, parameters, schema, version, data volume, or resource contention — changed.

### 1. The big picture
"The report is slow today" is a symptom, not a diagnosis. Without a baseline you can't tell whether the query changed, the data changed, the plan changed, or the machine is simply busy. Good tuning therefore begins with **understanding the problem**: which queries, how long, how often, and what resources they consume — and ends with a controlled change you can verify.

Regressions have a small set of root causes:

1. **Plan change** — the optimizer chose a different plan (new statistics, a recompile, a different first parameter).
2. **Data change** — volume or distribution shifted, so the old plan is no longer a good shape.
3. **Schema/index change** — an index was dropped, rebuilt, or added; a column type changed and a conversion is now implicit.
4. **Engine change** — an upgrade, new compatibility level or cardinality estimator, new optimizer fixes.
5. **Environment change** — memory pressure, parallelism settings, blocking, I/O latency, noisy neighbours.

The most valuable tool is a record of *what the query used to look like*. In SQL Server that's **Query Store**; in PostgreSQL, `pg_stat_statements` plus `auto_explain`; in MySQL, `performance_schema` digests and slow log; elsewhere, your APM/observability history.

### 2. Core concepts and vocabulary
- **Baseline** — recorded normal behaviour (duration, CPU, reads, rows, plan) for a query.
- **Regression** — a statistically meaningful worsening versus the baseline.
- **Query hash / fingerprint** — a normalised identifier for "the same query with different literals".
- **Plan hash / plan_id** — identifier of one particular plan shape; one query may have several.
- **Parameter sniffing** — the plan is compiled for the first parameter values seen and reused for all later ones.
- **Cardinality estimate** — predicted row count at each operator; large estimate-vs-actual gaps explain most bad plans.
- **Query Store** — SQL Server's per-database history of queries, plans and runtime statistics in time intervals.
- **Plan forcing** — telling the engine to keep using a specific known-good plan.
- **Wait statistics** — time the engine spent waiting on resources (locks, I/O, memory grants), the "why is it slow" view.
- **Logical reads** — pages read from the buffer pool; the most stable metric for comparing plan efficiency.

### 3. How it works, step by step
A repeatable investigation:

1. **Define the regression.** Which query/procedure, since when, how bad (elapsed, CPU, reads), for which parameters, for which users.
2. **Check the environment first.** CPU, memory, tempdb, I/O latency, blocking, a deployment or maintenance job at that time. If everything is slow, it's not one query.
3. **Find the query's history.** Compare current vs earlier runtime stats and plans (Query Store *Regressed Queries*, `pg_stat_statements` deltas).
4. **Compare plans.** Same shape? If different: what changed — join type, join order, index used, lookup vs scan, parallelism, memory grant? If the same: it's data volume, blocking or resources.
5. **Compare estimates with actuals** at the operator where the plan went wrong (look for a >10× gap).
6. **Name the cause**: stale/missing statistics, skewed parameter, non-sargable predicate, dropped index, new CE, etc.
7. **Mitigate** (force the good plan, update statistics, add `OPTION (RECOMPILE)`), then **fix** the root cause (index, rewrite, statistics strategy).
8. **Verify** against the baseline and monitor.

### 4. Worked example
A stored procedure `usp_OrdersByCustomer @CustomerID` went from 40 ms to 25 s overnight.

```sql
-- 1) Top consumers right now
SELECT TOP 10 qs.total_elapsed_time/qs.execution_count AS avg_elapsed_us,
       qs.execution_count, qs.total_logical_reads/qs.execution_count AS avg_reads,
       SUBSTRING(st.text,1,200) AS query_text
FROM   sys.dm_exec_query_stats qs
CROSS  APPLY sys.dm_exec_sql_text(qs.sql_handle) st
ORDER  BY qs.total_elapsed_time DESC;

-- 2) Query Store: plans per query and their average duration over time
SELECT q.query_id, p.plan_id, rs.avg_duration, rs.avg_logical_io_reads,
       rs.count_executions, rsi.start_time
FROM sys.query_store_query q
JOIN sys.query_store_plan p ON p.query_id = q.query_id
JOIN sys.query_store_runtime_stats rs ON rs.plan_id = p.plan_id
JOIN sys.query_store_runtime_stats_interval rsi ON rsi.runtime_stats_interval_id = rs.runtime_stats_interval_id
WHERE q.object_id = OBJECT_ID('dbo.usp_OrdersByCustomer')
ORDER BY rsi.start_time;
```

Finding: two plans. `plan_id 17` (index seek + key lookup, 40 ms) and `plan_id 42` (clustered index scan with hash join, 25 s). After last night's index rebuild the first execution used a customer with 2 million orders; the optimizer compiled a scan plan and reused it for customers with 3 orders.

Mitigation, then root cause:

```sql
-- quick mitigation: force the good plan
EXEC sp_query_store_force_plan @query_id = 101, @plan_id = 17;

-- real fix options
-- a) OPTION (RECOMPILE) for the skew-sensitive statement
-- b) OPTION (OPTIMIZE FOR (@CustomerID UNKNOWN))
-- c) SQL Server 2022 Parameter Sensitive Plan optimization (compat level 160)
-- d) split into two procedures for "big" and "small" customers
```

Verify with `SET STATISTICS IO, TIME ON` that logical reads are back near the baseline.

### 5. Under the hood
- **Why plans change.** A plan is cached after compilation and invalidated by schema changes, statistics updates crossing a threshold, `sp_recompile`, restart, memory pressure or plan-cache eviction. A recompile may compile for a different first parameter than before.
- **Cardinality estimation.** Histograms of up to 200 steps summarise a column's distribution; the density vector estimates the average rows per value. For parameters, the optimizer reads the histogram using the sniffed value; for local variables it uses an average, which hides skew. Changing compatibility level changes the CE model (new in 120, refined later), occasionally flipping plans.
- **Query Store** captures compile and runtime statistics aggregated by interval; `FORCE_LAST_GOOD_PLAN` automatic tuning (SQL Server 2017 Enterprise/Azure) can detect and revert regressions without humans.
- **Regressions without plan change.** Check blocking (`LCK_M_*`), memory grants (`RESOURCE_SEMAPHORE`), parallelism skew (`CXPACKET/CXCONSUMER`), I/O (`PAGEIOLATCH_*`), spills to tempdb in the plan warnings, and a cold cache after restart.
- **PostgreSQL equivalent.** Compare `EXPLAIN (ANALYZE, BUFFERS)` before/after; check `pg_stat_user_tables.last_analyze`, `autovacuum` lag, bloat, `plan_cache_mode` for prepared statements (generic vs custom plans), and extension-based plan capture (`auto_explain`, `pg_stat_statements`).

### 6. Performance and design implications
- Always capture **both** elapsed time and logical reads: time varies with load, reads reflect plan quality.
- Turn on Query Store (or an equivalent) *before* you need it; a baseline created after the incident is useless.
- Prefer root-cause fixes (statistics, indexing, rewrite) over permanent forced plans, which freeze a plan that may later become wrong and must be monitored.
- After upgrades or compatibility-level changes, test with the old and new CE using Query Store comparisons before cutover.
- Keep statistics maintenance deliberate: asynchronous auto-update stats on large tables, `FULLSCAN` or higher sampling for skewed columns.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ALTER DATABASE … SET QUERY_STORE = ON` | Starts capture of queries/plans/stats | Always on in production |
| Query Store *Regressed Queries* report | Lists queries slower than a baseline | First stop |
| `sp_query_store_force_plan` / `unforce_plan` | Pin/unpin a plan | Emergency mitigation |
| `SET STATISTICS IO, TIME ON` | Reads and CPU/elapsed per statement | Before/after verification |
| `sys.dm_exec_query_stats`, `sys.dm_exec_requests` | Aggregated/current runtime data | No Query Store available |
| `EXPLAIN (ANALYZE, BUFFERS)` | Actual plan with I/O in PostgreSQL | Plan comparison |
| `UPDATE STATISTICS … WITH FULLSCAN` | Refresh histogram precisely | Skewed or stale stats |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Force a plan to stop an incident | Data distribution is changing and the plan may rot |
| `OPTION (RECOMPILE)` | Statement is executed thousands of times per second |
| Update statistics (fullscan) | Very large table during peak hours |
| Add/adjust an index | It duplicates existing indexes or slows writes heavily |
| Roll back compat level / CE | You'd lose other upgrade benefits; use hints per query first |

### 9. Common mistakes
- Tuning without a baseline → can't prove improvement → record before changing.
- Looking only at the "estimated cost %" in plans → misleading → rely on actual rows, reads, time.
- Rebuilding all indexes as a first response → burns hours, discards cache → check the cause first.
- Forcing a plan and forgetting it → silent failures when index is dropped → monitor `force_failure_count`.
- Chasing one slow execution → it was blocking, not the plan → check waits.
- Testing on a tiny dev database → regression only appears with production-scale skew → use realistic data.

### 10. 🎯 Interview kit
**30-second answer:** I confirm the regression against a baseline, check the environment, compare the old and new plans (Query Store or `pg_stat_statements` + `EXPLAIN`), find where estimates diverge from actuals, identify the cause (stats, parameter sniffing, schema/version change), mitigate with a forced plan or recompile, then fix the root cause and verify.

**Structure a longer answer:** scope → environment → history → plan diff → estimates vs actuals → cause → mitigation → permanent fix → monitoring.

**Likely follow-ups:**
- *Same plan but slower?* → data volume, blocking, resource waits, cache cold.
- *Parameter sniffing options?* → recompile, optimize for unknown, PSP, split logic, hints.
- *What would you monitor to catch it early?* → plan-change count, p95 duration per query hash, reads per execution.
- *What if there's no Query Store?* → DMVs snapshots to a table on a schedule, APM traces.

**Signals interviewers look for:** a method not a guess, use of baselines, awareness that plan forcing is a bridge, not a cure.

### 11. Check yourself
- [ ] List the five root-cause categories of a regression.
- [ ] Write the Query Store query that shows plans and durations per time interval.
- [ ] Explain parameter sniffing with a skewed-customer example.
- [ ] Describe two ways a query can regress without its plan changing.
- [ ] State the risks of a forced plan.
<!-- explainer:end -->

## Practice questions

- [Basic] [How would you explain to a non-technical teammate why a report query is slow after looking at its plan?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Execution_Plans/Q-25145_How_would_you_explain_to_a_non-technical_teammate_why_a_repo.md>)
- [Intermediate] [How would you identify the slowest part of a query from its EXPLAIN ANALYZE output?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Execution_Plans/Q-25062_How_would_you_identify_the_slowest_part_of_a_query_from_its_.md>)
- [Intermediate] [How would you diagnose why a query that ran fast last week is now slow?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Execution_Plans/Q-25087_How_would_you_diagnose_why_a_query_that_ran_fast_last_week_i.md>)
- [Intermediate] [What tools or system views help you find the slowest queries running on a database?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Execution_Plans/Q-25110_What_tools_or_system_views_help_you_find_the_slowest_queries.md>)
- [Intermediate] [What is a slow query log and how do you use it to decide which queries need tuning?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Execution_Plans/Q-25112_What_is_a_slow_query_log_and_how_do_you_use_it_to_decide_whi.md>)
- [Intermediate] [What are the first three things you would check when someone reports a suddenly slow query in production?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Execution_Plans/Q-25146_What_are_the_first_three_things_you_would_check_when_someone.md>)
- [Intermediate] [Walk me through how you triaged a production query that suddenly regressed from 50ms to 8 seconds without any code change.](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Execution_Plans/Q-24950_Walk_me_through_how_you_triaged_a_production_query_that_sudd.md>)
- [Intermediate] [Explain how you use pg_stat_statements plus auto_explain together to catch and diagnose intermittent slow plans in production.](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Execution_Plans/Q-24955_Explain_how_you_use_pg_stat_statements_plus_auto_explain_tog.md>)

## Related concepts

- [Reading EXPLAIN](<../../SQL_Language/SQL_Fundamentals/Reading_EXPLAIN.md>) *(SQL Fundamentals)*
- [Index Tuning in Practice](<../Indexing/Index_Tuning_in_Practice/_index.md>) *(Indexing)*
- [Reading EXPLAIN ANALYZE](<../Indexing/Reading_EXPLAIN_ANALYZE.md>) *(Indexing)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Spotting Parameter Sniffing Issues
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Analyzing Performance Regressions
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Overview
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Identifying Plan Regressions
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Grouping and Summarizing Data › Thinking About Performance
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 4 Analyzing Query Behavior Using Execution Plans
- *Azure Data Engineering Cookbook* — Ahmad Osama — Analyzing Data with Azure Synapse Analytics › Getting ready
- *Data Engineering with Medallion Architecture* — Miki Eto — Scaling Data Platforms with Optimization › Process or steps

