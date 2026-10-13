---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Reading_EXPLAIN"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: ["MySQL"]
---

# Reading EXPLAIN

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** An execution plan shows the operators the engine chose and — with `EXPLAIN ANALYZE` / actual plans — how many rows each really produced versus what it estimated; reading it means finding where estimates diverge from reality, where rows or I/O balloon, and which operator costs the time.

### 1. The big picture
The optimizer turns your declarative SQL into a tree of physical operators (scans, seeks, joins, sorts, aggregates). Performance tuning is mostly: *look at the plan, find the expensive or surprising operator, understand why the optimizer picked it, and change the query, index or statistics so it picks better.*

There are two kinds of plan: **estimated** (what the optimizer expects; no execution) and **actual** (what happened, including real row counts, time and memory). The gap between them is the single most useful diagnostic: a plan is only as good as its cardinality estimates.

### 2. Core concepts and vocabulary
- **Operator / node** — one step: Seq/Table Scan, Index Seek/Scan, Nested Loops, Hash Join, Merge Join, Sort, Aggregate, Filter…
- **Cardinality estimate** — expected row count at each node.
- **Actual rows** — rows really produced (`EXPLAIN ANALYZE`, actual plan).
- **Cost** — optimizer's unitless estimate (I/O + CPU); only comparable within one query.
- **Seek vs scan** — navigate to a key range vs read everything.
- **Residual predicate / filter** — condition applied after rows are read (not used to seek).
- **Key/RID lookup** — fetching missing columns from the base table per index hit.
- **Spill** — sort/hash exceeded memory and wrote to disk (tempdb / temp files).
- **Statistics** — histograms/density the optimizer uses to estimate selectivity.
- **Plan cache / parameter sniffing** — cached plan built for first parameter values.

### 3. How it works, step by step
**Reading a plan**
1. Identify the plan flavor: PostgreSQL `EXPLAIN (ANALYZE, BUFFERS)`; MySQL `EXPLAIN ANALYZE`; SQL Server *Actual Execution Plan* / `SET STATISTICS IO, TIME ON`; Snowflake/BigQuery query profile.
2. Start at the **most expensive node** (highest % cost or time), then follow data flow: execution reads right → left / bottom → top in tree displays (leaves produce rows, parents consume).
3. Compare **estimated vs actual rows** at every node. A 10× or greater discrepancy is a red flag.
4. Check access paths: is a big table being scanned when a selective predicate exists?
5. Check join algorithm vs data volumes: nested loops over millions of rows, or hash join with a spill.
6. Look for **Sort**, **Spill**, **Key lookup**, **Filter** operators and warnings (implicit conversion, missing join predicate, missing statistics).
7. Hypothesize the cause (stale stats, non-sargable predicate, missing index, correlated columns, parameter sniffing), change one thing, re-measure.

**Typical shapes**

| You see | It means | Usual fix |
|---|---|---|
| Seq/Table Scan on large table with tiny result | No usable index or non-sargable predicate | Index, rewrite predicate |
| Nested Loops, inner side executed millions of times | Underestimated outer rows | Update stats, extended stats, hash join hint |
| Estimated 12 rows, actual 4,000,000 | Bad estimate (correlation/stale stats/skew) | `ANALYZE`, multi-column stats |
| Sort with spill | Memory grant too small | Index to provide order, increase memory, reduce rows |
| Key lookup × many | Non-covering index | `INCLUDE` columns |
| Filter after scan | Predicate not in seek | Reorder index, sargable rewrite |

### 4. Worked example
```sql
-- PostgreSQL
EXPLAIN (ANALYZE, BUFFERS)
SELECT o.customer_id, SUM(o.total)
FROM   orders o
JOIN   customers c ON c.id = o.customer_id
WHERE  c.country = 'IN' AND o.created_at >= now() - interval '30 days'
GROUP  BY o.customer_id;
```
Illustrative output (abridged):
```
HashAggregate  (rows=900 actual rows=41200)
  -> Hash Join  (rows=900 actual rows=185000)
       Hash Cond: (o.customer_id = c.id)
       -> Seq Scan on orders o (rows=60000 actual rows=2000000)
            Filter: (created_at >= ...)   Rows Removed by Filter: 18000000
       -> Hash -> Index Scan on customers c (rows=800 actual rows=52000)
            Index Cond: (country = 'IN')
Buffers: shared hit=120 read=410000   Execution Time: 9120 ms
```
How to read it: (a) `orders` is **seq-scanned** and 18 M rows are discarded by the filter → needs an index on `orders(created_at)` (or `(customer_id, created_at)`); (b) estimates are ~60× low at the join and customers (800 est vs 52,000 actual) → statistics are stale or `country` correlates with other columns: run `ANALYZE customers;` or `CREATE STATISTICS … (dependencies)`; (c) `read=410000` buffers shows it's I/O bound, not CPU. After indexing and analyzing, the plan flips to an index scan with fewer than 200 k rows read.

SQL Server equivalent: look for the thick arrows (row counts), warning icons on operators, **Actual vs Estimated Number of Rows**, **Number of Executions**, `Actual Logical Reads` in `STATISTICS IO`, and the *Memory Grant* / spill warnings in operator properties.

### 5. Under the hood
- **Plan search:** the optimizer enumerates join orders and access paths using a cost model fed by statistics, picks the cheapest, and stops after a time/complexity budget — so very complex queries get "good enough" plans.
- **Estimation:** selectivity for `col = v` comes from a histogram (most-common values + equi-depth buckets in PG; up to 200 steps in SQL Server). Multiple predicates are assumed *independent* by default (multiply selectivities), which underestimates when columns are correlated (city/country). Extended/multi-column statistics fix this.
- **Join algorithms:** nested loop cost ~ outer × inner-seek; hash join ~ build + probe (needs memory, can spill); merge join needs sorted inputs (cheap if indexes provide order). Wrong cardinality → wrong algorithm choice.
- **Parameter sniffing:** a cached plan compiled for a rare value can be disastrous for common values; options: `OPTION (RECOMPILE)`, `OPTIMIZE FOR`, Query Store plan forcing, or splitting code paths.
- **Operator timing vs cost:** cost percentages in estimated plans are guesses; in actual plans rely on measured time/IO per operator (PG `actual time`, SQL Server per-operator elapsed time).
- **`EXPLAIN ANALYZE` executes the query** — wrap DML in a transaction and roll back.

### 6. Performance and design implications
- Most tuning wins: add/adjust an index (leftmost prefix, covering), make predicates sargable, fix statistics, remove unnecessary `DISTINCT`/sorts, avoid `SELECT *`.
- `DISTINCT` forces a sort or hash aggregate; often it masks a join fan-out (see Join Fan-out).
- Prefer set-based and early filtering/aggregation; push predicates down.
- Keep stats fresh (auto-update, `ANALYZE` after bulk loads); use histograms on skewed columns.
- Measure before/after with the same data; compare logical reads and elapsed time, not just "feels faster".
- Beware over-indexing: each index adds write cost — verify with usage statistics.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `EXPLAIN` / `EXPLAIN (ANALYZE, BUFFERS)` | Plan / plan with real execution | Diagnose slow query |
| `SET STATISTICS IO, TIME ON` | I/O and time per statement (SQL Server) | Compare variants |
| Actual execution plan (SSMS) | Real rows, warnings | Find estimation gaps |
| `ANALYZE` / `UPDATE STATISTICS` | Refresh stats | After bulk change |
| `CREATE STATISTICS … (dependencies, ndistinct)` | Extended stats (PG) | Correlated columns |
| Query Store / `pg_stat_statements` | Historical query cost | Find top offenders |
| `OPTION (RECOMPILE)` | Fresh plan per run | Parameter sniffing |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Estimated plan only | You need real row counts (it lies when stats are wrong) |
| `EXPLAIN ANALYZE` | Query modifies data or is too slow to execute fully |
| Plan hints | As a permanent fix instead of correcting stats/indexes |
| New index | Table is write-heavy and index would be rarely used |
| `RECOMPILE` | Extremely frequent query (compile cost) |

### 9. Common mistakes
- Tuning from estimated plan cost % → chasing wrong operator → use actual rows/time.
- Ignoring estimate vs actual gaps → wrong join algorithm persists → fix statistics.
- Adding indexes blindly → write penalty → check usage and workload.
- Forcing hints permanently → plan rots as data changes → fix root cause.
- Running `EXPLAIN ANALYZE` on `DELETE` in prod → data gone → wrap in `BEGIN … ROLLBACK`.
- Comparing cost numbers across different queries → meaningless → compare time/IO.

### 10. 🎯 Interview kit
**30-second answer:** I run `EXPLAIN ANALYZE` / get the actual plan, find the node where time or I/O concentrates, compare estimated and actual rows to spot bad statistics, and check access paths and join algorithms. Then I fix the root cause — index, sargable rewrite, statistics, or query shape — and re-measure.
**Structure a longer answer:** collect actual plan → locate hot operator → check cardinality gaps → identify cause → apply change → verify → guard against regression (Query Store, monitoring).
**Likely follow-ups:**
- *Estimated 12, actual 4M rows — why?* → stale/missing stats, correlated predicates, skew, parameter sniffing; fix with stats/extended stats.
- *Seek vs scan?* → seek navigates to a range; scan reads whole structure.
- *What does DISTINCT cost?* → sort/hash on the full row set; check whether it hides fan-out.
- *Why did the plan change overnight?* → stats update, parameter sniffing, data growth.
**Signals interviewers look for:** systematic method, focus on cardinality, knowledge of operators, measurement discipline.

### 11. Check yourself
- [ ] Explain estimated vs actual plans and when each is useful.
- [ ] Given a plan fragment, identify the scan, the residual filter and the bad estimate.
- [ ] Name three causes of cardinality misestimates and a fix for each.
- [ ] Describe how each join algorithm's cost scales.
- [ ] Explain how to safely run `EXPLAIN ANALYZE` on an `UPDATE`.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../../../TOOLS/MySQL/_index.md>)

## Practice questions

- [Basic] [What does the DISTINCT keyword do and how does it affect performance?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40242_What_does_the_DISTINCT_keyword_do_and_how_does_it_affect_per.md>)
- [Intermediate] [What is denormalization and when would you deliberately denormalize?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40077_What_is_denormalization_and_when_would_you_deliberately_deno.md>)
- [Intermediate] [What is SQL injection and how do you prevent it?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40953_What_is_SQL_injection_and_how_do_you_prevent_it.md>)
- [Intermediate] [Why is a p99 latency more useful than an average when monitoring service health, and how do you compute it in SQL?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40982_Why_is_a_p99_latency_more_useful_than_an_average_when_monito.md>)
- [Deep] [An EXPLAIN ANALYZE shows a rows estimate of 12 but 4 million actual on a join. What causes such a misestimate and which statistics or extended-stats objects would you create to correct it?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39871_An_EXPLAIN_ANALYZE_shows_a_rows_estimate_of_12_but_4_million.md>)
- [Deep] [What is a full table scan and when is it acceptable versus a red flag?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40931_What_is_a_full_table_scan_and_when_is_it_acceptable_versus_a.md>)
- [Expert] [Your report query aggregates millions of rows and times out. How would you make it faster?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39789_Your_report_query_aggregates_millions_of_rows_and_times_out_.md>)
- [Expert] [What is an execution plan, and how do you read a nested loop vs hash join vs merge join?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40919_What_is_an_execution_plan_and_how_do_you_read_a_nested_loop_.md>)

## Related concepts

- [Reading EXPLAIN ANALYZE](<../../Query_Performance/Indexing/Reading_EXPLAIN_ANALYZE.md>) *(Indexing)*
- [Statistics and Sargability](<../../Query_Performance/Execution_Plans/Statistics_and_Sargability.md>) *(Execution Plans)*
- [Filtered Indexes and Plan Cache](<../../../TOOLS/SQL_Server/T-SQL/Filtered_Indexes_and_Plan_Cache.md>) *(T-SQL)*

## References

- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Wait Statistics
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Impact Analysis
- *The Enterprise Big Data Lake* — Alex Gorelik — Introduction to Big Data and Data Science › Explainability
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Advanced Operator Analysis
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 10. Batch Processing
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — Data Quality Isn't About Pristine Data › Analysts
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Table Deletes, Updates, and Merges › Analyzing the MERGE operation with DESCRIBE HISTORY

