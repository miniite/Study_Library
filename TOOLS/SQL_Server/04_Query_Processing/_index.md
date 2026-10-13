---
id: "TOOLS/SQL_Server/04_Query_Processing"
level: 3
type: "curriculum"
curriculum: "sql_server"
---

# Query Processing and Tuning

**L3 Practitioner** · [↑ SQL Server](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Every query you send is parsed, bound, optimized into an execution plan, cached, and then executed. Query tuning means measuring that pipeline, finding the few queries that matter, and changing the plan, the query or the indexes until the numbers improve.

### 1. The big picture
The earlier blocks covered how SQL Server stores data (pages, B-trees, indexes) and how it summarizes data (statistics). This block covers what happens **between** "you press Execute" and "rows come back". It is the part of the engine you interact with most when you tune performance.

Two big components handle a query:
- The **relational engine** (also called the query processor) parses the text, works out what it means, chooses a plan and runs it.
- The **storage engine** reads and writes pages, takes locks and manages the buffer pool. The relational engine asks it for rows.

Almost every performance problem shows up as a bad decision in the relational engine: a scan where a seek was possible, a nested loop that runs a million times, a plan compiled for one parameter value and reused for a very different one, or a cursor that processes rows one at a time. The lessons in this block go through each stage and each common failure:

| Child lesson | Question it answers |
|---|---|
| Query Optimizer | How does SQL Server choose among thousands of possible plans? |
| Execution Plans | How do I read the plan and find the expensive part? |
| Plan Cache and Reuse | Where are plans kept, and when are they reused? |
| Recompilation | What forces a new plan, and when is that good or bad? |
| Parameter Sensitivity | Why is the same procedure fast for one value and slow for another? |
| Query Store | How do I see plan history and force a known-good plan? |
| Automatic Tuning | Can SQL Server fix plan regressions itself? |
| Intelligent Query Processing | Which features adapt plans automatically (memory grant feedback, adaptive joins and others)? |
| Query Design | Which ways of writing T-SQL help or hurt the optimizer? |
| Row-by-Row Processing | Why are cursors and loops slow, and how do I rewrite them as set-based code? |
| Tuning Methodology | What process do I follow from "it's slow" to "it's fixed and verified"? |

### 2. Core concepts and vocabulary
- **Batch**: one or more T-SQL statements sent to the server together (in SSMS, everything between two `GO` lines).
- **Parsing**: checking that the syntax is valid. Produces a **parse tree**.
- **Binding (algebrizer)**: resolving names (tables, columns, types) and building a **query processor tree**. It also adds implicit conversions.
- **Query optimizer**: the cost-based component that turns the processor tree into an execution plan.
- **Execution plan**: the step-by-step recipe (operators like Index Seek, Hash Match, Sort) the engine follows. It is also the best window into the optimizer's choices.
- **Estimated vs actual plan**: there is really only one plan. An "actual" plan is the same plan with **runtime metrics** added (actual rows, executions, time, memory used).
- **Cost**: a unitless number the optimizer uses to compare plans. It is an estimate, not a measurement of time.
- **Plan cache**: memory where compiled plans are stored so the next execution can skip optimization.
- **Recompile**: throwing away a cached plan and optimizing again.
- **Parameter sniffing**: the optimizer builds the plan using the parameter values of the first call; later calls reuse that plan.
- **Query Store**: a per-database feature that saves queries, plans and runtime statistics to disk over time.
- **Baseline / comparison point**: a measurement taken before a change, so you can prove the change helped.

### 3. How it works, step by step
1. **Parse.** The text is checked for syntax. A single error anywhere in a batch stops the whole batch before any optimization. (A typo such as `FORM` instead of `FROM` in the last statement means even the valid `CREATE TABLE` at the top never runs.)
2. **Bind.** Object names are resolved, data types identified, aggregates mapped, and implicit conversions added. The algebrizer may also rewrite syntax: a `BETWEEN 62500 AND 62550` predicate shows up in the plan as `>= 62500 AND <= 62550`.
3. **Check the plan cache.** If a matching plan already exists, it is reused and steps 4–5 are skipped. This is why compiling once and reusing often matters so much.
4. **Optimize.** The optimizer simplifies the tree (removes redundant joins, folds constants, uses constraints), checks whether a **trivial plan** is enough, and if not runs cost-based optimization phases. It uses statistics to estimate row counts and picks the cheapest plan it finds in a reasonable time. It also decides whether the plan should run in parallel.
5. **Cache.** The plan is stored in the plan cache. Plans age out under memory pressure; expensive plans stay longer.
6. **Execute.** The relational engine runs the operators, pulling rows from the storage engine. Runtime numbers (CPU, reads, duration, rows) are recorded in DMVs such as `sys.dm_exec_query_stats` and, if enabled, in Query Store.

```
T-SQL text ─► Parse ─► Bind (algebrizer) ─► Plan in cache? ── yes ──► Execute
                                               │ no
                                               ▼
                                    Optimize (cost-based) ─► Cache plan ─► Execute
```

### 4. Worked example
Measure first, then look at the plan.

```sql
-- 1. Find the queries that matter most right now (from the plan cache)
SELECT TOP (10)
       qs.execution_count,
       qs.total_elapsed_time / 1000      AS total_ms,
       qs.total_logical_reads,
       SUBSTRING(st.text, qs.statement_start_offset/2 + 1,
         (CASE qs.statement_end_offset WHEN -1 THEN DATALENGTH(st.text)
               ELSE qs.statement_end_offset END - qs.statement_start_offset)/2 + 1) AS statement_text
FROM sys.dm_exec_query_stats AS qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) AS st
ORDER BY qs.total_elapsed_time DESC;

-- 2. Take a comparison point for one candidate
SET STATISTICS IO, TIME ON;
SELECT CustomerID, SalesOrderID,
       ROW_NUMBER() OVER (PARTITION BY CustomerID ORDER BY OrderDate) AS RowNumber
FROM Sales.SalesOrderHeader;
SET STATISTICS IO, TIME OFF;
-- Turn on the actual plan first (Ctrl+M in SSMS)
```
What to read:
- In the DMV output, sort by **total** cost (elapsed time × executions), not just the slowest single run. A 5 ms query run 2 million times an hour matters more than a 2-second report run once a day.
- In the Messages tab, note **logical reads** and **CPU time**. These are your "before" numbers.
- In the plan, the window function produces **Segment** and **Sequence Project** operators (harmless) and a **Sort** that takes most of the cost. The Sort is the part you can fix: an index on `(CustomerID, OrderDate)` that includes `SalesOrderID` delivers rows already in order, and the Sort disappears. Re-run and compare reads and CPU with the "before" numbers.

### 5. Under the hood
- The optimizer does not search every possible plan. For a query with many joins the number of possible join orders grows very quickly, so it works in **phases** (trivial plan → search 0 → search 1 → search 2) and stops when it finds a plan that is "good enough" for the time it has spent. The plan property *Reason For Early Termination* shows why it stopped (for example *Good Enough Plan Found* or *Time Out*).
- Optimization costs mostly CPU. That is why plan reuse matters, and why too many recompiles show up as high CPU.
- Plans are the same no matter where you capture them (DMVs, Extended Events, Query Store, SSMS). For the same query without a recompile, they are identical. Only the runtime metrics differ.

### 6. Performance and design implications
- **Tune the queries that matter.** Pick targets by total resource use and business importance. Shaving 1 ms off a query called a few times an hour is wasted time.
- **Always have a comparison point:** DMV numbers, application round-trip time, `STATISTICS IO/TIME`, or Query Store history. Without "before" and "after" you cannot prove anything improved.
- **Tuning is iterative.** Data grows, code changes and new versions change the optimizer, so the cycle is measure → find → fix → validate → repeat.
- Cost percentages in a plan are **estimates**. Use them to find where to look, then confirm with actual rows, reads and time.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SET STATISTICS IO, TIME ON` | Prints reads per table and CPU/elapsed time per statement | Before/after comparison of one query |
| Ctrl+M / Ctrl+L in SSMS | Actual plan (runs the query) / estimated plan (doesn't) | Reading plans |
| `sys.dm_exec_query_stats` | Cumulative stats for cached plans | Finding top resource consumers |
| `sys.dm_exec_requests` | What is running right now | Live troubleshooting |
| `sys.dm_exec_query_plan(plan_handle)` | Returns the cached plan XML | Getting the plan for a DMV row |
| `ALTER DATABASE … SET QUERY_STORE = ON` | Enables plan and runtime history | Every production database (on by default for new databases since SQL Server 2022) |
| `OPTION (RECOMPILE)` | Compiles a fresh plan every run | Parameter-sensitive queries that run rarely |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Plan reuse (parameterized queries, procedures): high-frequency OLTP queries | Data skew makes one plan bad for some values (parameter sensitivity) |
| Recompiling every time: rare, expensive queries with very different parameter values | Queries running thousands of times a second; compile CPU adds up |
| Plan forcing (Query Store): a known-good plan regressed | As a permanent fix; forced plans can become wrong as data changes |
| Rewriting the query or adding an index | Without measuring first; every index slows writes |

### 9. Common mistakes
- Treating the "estimated cost %" as real time → tuning the wrong operator → compare actual rows and reads instead.
- Tuning without a baseline → no proof of improvement, and sometimes a hidden regression → capture numbers before every change.
- Testing a data-modification query on a stressed production server → side effects and load → test on a copy.
- Running `DBCC FREEPROCCACHE` on production to "fix" a slow query → every plan recompiles, CPU spikes → remove one plan by its `plan_handle`, or fix the root cause.
- Rewriting set-based code as loops "for clarity" → runs row by row, orders of magnitude slower → keep it set-based.

### 10. 🎯 Interview kit
**30-second answer:** "A query goes through parsing, binding, optimization and execution. The cost-based optimizer uses statistics to estimate row counts and picks the cheapest plan it can find in a limited time, then caches it for reuse. When I tune, I first find the top consumers from DMVs or Query Store, take baseline numbers, read the actual plan to find where estimates and reality diverge, fix the cause (index, statistics, query rewrite or plan stability) and measure again."
**Structure a longer answer:** clarify the symptom (always slow or sometimes slow?) → show the pipeline → explain how estimates drive the plan → trade-offs of reuse vs recompile → close with a real case, such as a Sort removed by an index that matches the window's PARTITION BY/ORDER BY.
**Likely follow-ups:** *Estimated vs actual plan?* → same plan; actual adds runtime metrics. *Why might a procedure suddenly get slow?* → a recompile sniffed a different parameter, statistics changed, or a plan was evicted. *How do you find the worst queries?* → `sys.dm_exec_query_stats` ordered by total CPU/reads/duration, or Query Store's Top Resource Consuming Queries report.
**Signals interviewers look for:** measuring before changing; prioritizing by total impact; knowing cost is an estimate; understanding plan reuse and its risks.

### 11. Check yourself
- [ ] Draw the pipeline from T-SQL text to executed plan and say which step a syntax error stops at.
- [ ] Explain why "estimated" and "actual" plans are the same plan.
- [ ] Write the DMV query that lists the top 10 statements by total elapsed time.
- [ ] Explain why a 5 ms query might be a better tuning target than a 30-second one.
- [ ] Remove the Sort from a `ROW_NUMBER() OVER (PARTITION BY … ORDER BY …)` query with an index, and prove it with logical reads.
<!-- explainer:end -->

## In this section (study in order)

1. [The Query Optimizer](<Query_Optimizer/_index.md>) (L4 Advanced)
2. [Reading Execution Plans](<Execution_Plans/_index.md>) (L4 Advanced)
3. [Plan Cache and Plan Reuse](<Plan_Cache_and_Reuse/_index.md>) (L4 Advanced)
4. [Parameter Sniffing and Sensitivity](<Parameter_Sensitivity/_index.md>) (L4 Advanced)
5. [Recompilation](<Recompilation/_index.md>) (L4 Advanced)
6. [Query Store](<Query_Store/_index.md>) (L4 Advanced)
7. [Intelligent Query Processing](<Intelligent_Query_Processing/_index.md>) (L4 Advanced)
8. [Automatic Tuning](<Automatic_Tuning/_index.md>) (L4 Advanced)
9. [Query Design for Performance](<Query_Design/_index.md>) (L4 Advanced)
10. [Row-by-Row vs Set-based](<Row_by_Row_Processing/_index.md>) (L4 Advanced)
11. [A Query Tuning Methodology](<Tuning_Methodology/_index.md>) (L4 Advanced)

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Conclusion
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 1 Query Performance Tuning
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Structure
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 21 Intelligent Query Processing
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 3 Methods For Capturing Query Performance Metrics
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 2 Execution Plan Generation And The Query Optimizer
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 15: Case Studies and the Future of T-SQL › Understanding the Performance Problem
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Introduction
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Using Execution Plans

