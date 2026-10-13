---
id: "01_Databases_and_SQL/Query_Performance/Execution_Plans/Reading_Execution_Plans"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Query_Performance/Execution_Plans"
tools: ["MySQL"]
---

# Reading Execution Plans

**L4 Advanced** · [↑ Execution Plans](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** An execution plan is the tree of physical operators the engine uses to run a query; you read it to see *how* rows are found, joined and sorted, and — by comparing estimated with actual rows — *why* it is slow.

### 1. The big picture
SQL says *what* you want; the **query optimizer** decides *how* to get it. It enumerates alternatives (which index, which join algorithm, which order), assigns each a cost from statistics and a cost model, and picks the cheapest it finds. The result is the execution plan: a tree of operators through which rows flow.

A plan is the single best window into the optimizer's choices — scans vs seeks, join methods, sorts, spills, parallelism — but it is **not** a measure of performance by itself. Cost percentages are *estimates* even in an "actual" plan. You judge a plan by actual rows, actual reads and actual time, and by how far those differ from the optimizer's expectations.

Every engine exposes plans: `EXPLAIN` in PostgreSQL/MySQL/Snowflake/Spark, graphical and XML plans in SQL Server, "Query Profile" in Snowflake and Databricks. The vocabulary differs but the ideas are the same.

### 2. Core concepts and vocabulary
- **Operator** — a node: scan, seek, join, sort, aggregate, spool, exchange…
- **Estimated vs actual plan** — estimated is produced without running (cheap, no runtime numbers); actual includes real row counts, executions, memory grants, spills, wait stats.
- **Table/index scan** — read all pages (or all index leaf pages) of the object.
- **Index seek** — navigate the B-tree to a key or range.
- **Key/RID lookup** — fetch non-indexed columns from the base row for each index hit.
- **Nested loops / hash / merge join** — three join algorithms (inner side repeatedly probed; hash table built on one input; two sorted inputs merged).
- **Sort / Hash aggregate / Stream aggregate** — ordering and grouping operators; sorts and hashes need memory.
- **Spill** — an operator ran out of its memory grant and wrote to tempdb/disk.
- **Parallelism (exchange/gather)** — operators run on multiple threads with repartitioning.
- **Cardinality** — number of rows an operator produces; estimated rows (ER) vs actual rows (AR).
- **Cost** — optimizer's unitless estimate of resource consumption.

### 3. How it works, step by step
**Reading a plan**
1. **Know the flow.** Data flows from the leaves (tables/indexes) up to the root (SELECT). Graphical SQL Server plans are drawn right-to-left; PostgreSQL text plans are indented, with children below parents. Execution is pull-based from the root.
2. **Start at the leaves.** How are tables accessed: seek or scan, which index, with what predicate (Seek Predicate vs residual Predicate)?
3. **Follow arrows.** Thicker arrows (actual rows) mean more data; note where it suddenly becomes large or small.
4. **Examine joins.** Which algorithm and which side is outer/inner? Nested loops with a large outer input and a lookup inner side is a classic bad shape.
5. **Check estimates vs actuals** at every operator. The first place they diverge sharply is usually the root cause.
6. **Look for warnings and expensive items:** scans of big tables, key lookups with high execution counts, sorts, spills, implicit-conversion warnings, missing-index hints, "no join predicate".
7. **Cross-check** with I/O: `SET STATISTICS IO ON` (logical reads), `STATISTICS TIME`, or `BUFFERS` in PostgreSQL.

### 4. Worked example
```sql
-- PostgreSQL
EXPLAIN (ANALYZE, BUFFERS)
SELECT c.name, SUM(o.total)
FROM   customers c JOIN orders o ON o.customer_id = c.id
WHERE  o.created_at >= '2026-01-01' AND c.country = 'DE'
GROUP  BY c.name;
```

Typical output (abridged):

```
HashAggregate  (cost=4120..4130 rows=40) (actual time=210..211 rows=1830 loops=1)
  -> Hash Join (cost=95..3900 rows=44000) (actual time=3.1..180 rows=612000 loops=1)
       Hash Cond: (o.customer_id = c.id)
       -> Index Scan using ix_orders_created on orders o
            (rows=410000) (actual rows=1900000 loops=1)  Buffers: shared hit=21000 read=18000
       -> Hash -> Seq Scan on customers c (rows=700) (actual rows=1830)
                   Filter: (country = 'DE')
Planning Time: 0.4 ms   Execution Time: 212 ms
```

How to read it: the leaf `Index Scan` estimated 410k rows but returned 1.9M — a ~5× miss that propagates upward (the join estimate 44,000 vs actual 612,000 and the aggregate 40 vs 1,830). The fix is not at the join: refresh statistics on `orders.created_at` or add extended statistics, and the planner might choose a sequential scan with a hash join, which suits 1.9M rows far better than an index scan with random I/O.

Equivalent SQL Server tools:

```sql
SET STATISTICS IO, TIME ON;
-- Ctrl+M (SSMS) "Include Actual Execution Plan", or:
SET STATISTICS XML ON;
-- Estimated only:
SET SHOWPLAN_XML ON;
```

### 5. Under the hood
- **Optimization phases.** The optimizer parses and binds the query, normalises it into a logical tree, explores equivalent logical/physical alternatives with rules (join reorderings, predicate pushdown, index choice), costs them with statistics, and stops at "good enough" (SQL Server has *trivial plan*, *transaction processing*, and *full optimization* stages with a time budget).
- **Join algorithm choice.** Nested loops are cheap when the outer input is tiny and the inner side can seek. Hash join is a good general-purpose choice for large unsorted inputs but needs a memory grant. Merge join excels when both inputs are already sorted on the join key (e.g. by an index).
- **Memory grants.** Sorts and hashes get a memory grant sized from *estimated* rows. Underestimates cause spills; overestimates waste memory and queue other queries (`RESOURCE_SEMAPHORE`).
- **Estimated plan vs actual.** An estimated plan is what the engine *would* do; it can differ from what happens because of parameter values, adaptive operators (adaptive joins, batch mode), and runtime re-optimisation (Spark AQE). "Actual" plans add `ActualRows`, `ActualExecutions`, `ActualElapsedms`, and since 2016 SP1 *lightweight profiling* lets you pull the actual plan of a running query via `sys.dm_exec_query_statistics_xml`.
- **Other engines.** MySQL's `EXPLAIN` columns: `type` (ALL = full scan, ref, range, const), `key`, `rows`, `Extra` (`Using filesort`, `Using temporary`). Spark `explain()` shows parsed → analyzed → optimized logical plan and the physical plan; look for `BroadcastHashJoin` vs `SortMergeJoin` and `Exchange` (shuffle) nodes.
- **Cost is relative.** "Cost 85%" next to an operator means 85% of the *estimated* total. If estimates are wrong, so is the percentage — so never optimise by percentage alone.

### 6. Performance and design implications
- Big scans on large tables, high-execution-count key lookups, and spills are the first things to check.
- A scan is not automatically bad: for a query returning a large fraction of a table, a scan beats thousands of random lookups.
- Warnings are cheap wins: implicit conversions, missing statistics, no join predicate (accidental cross join).
- Observe time as well as plan: operator "actual elapsed" shows where wall-clock goes, which cost cannot.
- Capture plans in production safely: Query Store, `auto_explain` (PostgreSQL), slow-query log with `EXPLAIN`.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `EXPLAIN` / `EXPLAIN (ANALYZE, BUFFERS)` | Estimated / actual plan with I/O | PostgreSQL tuning |
| `EXPLAIN FORMAT=TREE` / `EXPLAIN ANALYZE` (MySQL 8) | Tree plan with timing | MySQL |
| `SET SHOWPLAN_XML ON` | Estimated plan, query not run | Safe look at heavy queries |
| `SET STATISTICS XML/IO/TIME ON` | Actual plan, reads, time | SQL Server tuning |
| `EXPLAIN (GENERIC_PLAN)` (PostgreSQL 16) | Plan with `$1` params, no values | Prepared statements |
| `df.explain("formatted")` (Spark) | Physical plan details | Spark SQL |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Estimated plan | You need row counts/time or the query is parameter-sensitive |
| Actual plan | The query is destructive or very long-running (EXPLAIN ANALYZE *executes* it; wrap DML in a rolled-back transaction) |
| Trust the optimizer, fix stats | You're tempted to hint every query |
| Graphical plan | Plan is huge — use XML/text and search |

### 9. Common mistakes
- Running `EXPLAIN ANALYZE` on an `UPDATE` in production → it really updates → wrap in `BEGIN … ROLLBACK`.
- Reading only cost % → fix the wrong operator → compare estimated vs actual rows.
- Ignoring the "Predicate" vs "Seek Predicate" difference → residual filtering scans many rows → ensure the filter column is in the key.
- Comparing plans on cold vs warm caches → misleading → compare logical reads.
- Assuming a scan means missing index → may be optimal for large results → check selectivity.

### 10. 🎯 Interview kit
**30-second answer:** The plan is a tree of physical operators the optimizer chose using statistics. I read from the leaves up, look at access paths, join types and row counts, and compare estimated with actual rows to find where the optimizer was wrong; I confirm with logical reads and time.

**Structure a longer answer:** what a plan is → how to obtain estimated/actual → read order → key operators → diagnosing misestimates and spills → verifying a fix.

**Likely follow-ups:**
- *Estimated vs actual plan?* → no execution vs real counts, spills and timings.
- *When does the optimizer choose hash over nested loops?* → large unsorted inputs, no useful inner index.
- *What does `ANALYZE` add?* → real execution with timing; careful with DML.
- *What would you do with a key lookup executed a million times?* → covering index or rewrite.

**Signals interviewers look for:** structured reading, scepticism about cost %, use of actuals.

### 11. Check yourself
- [ ] Describe the data-flow direction in a plan and where you start reading.
- [ ] Contrast nested loops, hash and merge joins and when each wins.
- [ ] Explain why a memory grant can cause a spill.
- [ ] Obtain an actual plan in PostgreSQL and SQL Server.
- [ ] Say why cost % can mislead.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../../../TOOLS/MySQL/_index.md>)

## Practice questions

- [Basic] [How do you view the execution plan for a query in PostgreSQL using EXPLAIN?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Execution_Plans/Q-25070_How_do_you_view_the_execution_plan_for_a_query_in_PostgreSQL.md>)
- [Basic] [What is the purpose of the ANALYZE keyword in EXPLAIN ANALYZE and how does it differ from plain EXPLAIN?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Execution_Plans/Q-25081_What_is_the_purpose_of_the_ANALYZE_keyword_in_EXPLAIN_ANALYZ.md>)
- [Basic] [How do you read the execution plan of a query in MySQL and what does EXPLAIN output show?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Execution_Plans/Q-25102_How_do_you_read_the_execution_plan_of_a_query_in_MySQL_and_w.md>)
- [Intermediate] [In a MySQL EXPLAIN what does the 'type' column (e.g. ALL/index/ref/const) tell you about access method?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Execution_Plans/Q-25104_In_a_MySQL_EXPLAIN_what_does_the_type_column_e_g_ALL_index_r.md>)
- [Intermediate] [What does the 'rows' column in a MySQL EXPLAIN represent and why is it only an estimate?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Execution_Plans/Q-25105_What_does_the_rows_column_in_a_MySQL_EXPLAIN_represent_and_w.md>)
- [Intermediate] [What does 'Using index' in a MySQL EXPLAIN Extra column signify?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Execution_Plans/Q-25108_What_does_Using_index_in_a_MySQL_EXPLAIN_Extra_column_signif.md>)
- [Intermediate] [What does the EXPLAIN option FORMAT JSON or FORMAT TEXT change about the plan output?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Execution_Plans/Q-25141_What_does_the_EXPLAIN_option_FORMAT_JSON_or_FORMAT_TEXT_chan.md>)
- [Intermediate] [Explain how you would use EXPLAIN (GENERIC_PLAN) or plan-only tooling to tune a query without executing it.](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Execution_Plans/Q-25034_Explain_how_you_would_use_EXPLAIN_GENERIC_PLAN_or_plan-only_.md>)

## Related concepts

- [Reading EXPLAIN ANALYZE](<../Indexing/Reading_EXPLAIN_ANALYZE.md>) *(Indexing)*
- [Reading EXPLAIN](<../../../TOOLS/MySQL/MySQL_Platform/Reading_EXPLAIN.md>) *(MySQL Platform)*
- [Reading EXPLAIN](<../../SQL_Language/SQL_Fundamentals/Reading_EXPLAIN.md>) *(SQL Fundamentals)*

## References

- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Chapter 8. Queries, Modeling, and Transformation › Use the explain plan and understand your query's performance
- *Databricks Certified Associate Developer for Apache Spark* — Saba Shah — Explain the optimized query plan › Explain the optimized query plan \

