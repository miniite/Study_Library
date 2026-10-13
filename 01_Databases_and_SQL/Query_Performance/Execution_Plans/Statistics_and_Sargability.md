---
id: "01_Databases_and_SQL/Query_Performance/Execution_Plans/Statistics_and_Sargability"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Query_Performance/Execution_Plans"
tools: []
---

# Statistics and Sargability

**L4 Advanced** · [↑ Execution Plans](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** The optimizer chooses plans from *statistics* (summaries of how values are distributed) and can only use an index seek when the predicate is *sargable*; stale statistics or non-sargable predicates are the two most common reasons good indexes go unused.

### 1. The big picture
Two independent things decide whether an index helps your query:

1. **Can** the engine use it? That's *sargability*: is the predicate shaped so the index's sort order can be exploited?
2. **Will** the optimizer choose it? That's *statistics*: does it believe the predicate is selective enough that seeking beats scanning?

Both can fail silently. A perfectly indexed column wrapped in `UPPER()` can never seek. A perfectly sargable predicate can still produce a bad plan if the histogram says "5 rows" when there are 5 million. The optimizer isn't guessing randomly; it estimates the number of rows each operator will produce (**cardinality**) from statistics, then costs alternatives. Garbage statistics in, garbage plan out.

### 2. Core concepts and vocabulary
- **Statistics object** — metadata on one or more columns, usually tied to an index or auto-created for a predicate column.
- **Histogram** — up to 200 steps (SQL Server) of the leading column: for each range, the upper bound, rows equal to it, rows in the range, distinct count.
- **Density vector** — 1 / number of distinct values for each prefix of a multi-column statistic; gives the average rows per value.
- **Selectivity** — fraction of rows a predicate returns (low = few rows = selective).
- **Cardinality estimation (CE)** — computing expected rows from selectivity and statistics.
- **Auto-update statistics** — refresh triggered when modifications exceed a threshold; may be synchronous or asynchronous.
- **Sampling** — fraction of rows read to build the histogram; `FULLSCAN` reads all.
- **Sargable predicate** — `column op constant/param`, where op is `=`, `<`, `>`, `BETWEEN`, `LIKE 'abc%'`, `IN`, `IS NULL`.
- **Residual predicate** — a filter applied after rows are found, not used to navigate the index.
- **Ascending-key problem** — newest values lie beyond the last histogram step, so they're estimated as ~1 row.

### 3. How it works, step by step
**Estimation for `WHERE OrderDate = '2026-03-14'`**
1. The optimizer finds a statistic whose leading column is `OrderDate`.
2. It searches the histogram for the value. If it equals a step's upper bound: use `EQ_ROWS`. If inside a step's range: use `AVG_RANGE_ROWS`. If beyond the last step: assume a minimal estimate (or, in newer CE, extrapolate).
3. It multiplies selectivities for multiple predicates (CE 70: independence assumption; CE 120+: *exponential backoff* — a softer assumption).
4. It compares: seek + lookups at *N* rows vs scan; picks cheaper.
5. At runtime, actual rows may differ; if too far, you see spills, bad join choices, wrong parallelism.

**Sargability test**
1. Does the indexed column appear *alone* on one side of the comparison?
2. Are the types equal (or is the conversion applied to the value, not the column)?
3. For `LIKE`, is the wildcard only at the end?
4. If yes to all → seek possible. Otherwise → scan or index scan with residual filter.

### 4. Worked example
```sql
-- Inspect the histogram
DBCC SHOW_STATISTICS ('Sales.SalesOrderHeader', 'IX_SalesOrderHeader_OrderDate');
-- Output sections: header (rows, rows sampled, last updated),
--                  density vector, histogram (RANGE_HI_KEY, RANGE_ROWS, EQ_ROWS, ...)

-- Non-sargable
SELECT COUNT(*) FROM Sales.SalesOrderHeader
WHERE  YEAR(OrderDate) = 2013;
--   Index Scan, estimate: ~30% guess (fixed fraction), reads ≈ whole index

-- Sargable
SELECT COUNT(*) FROM Sales.SalesOrderHeader
WHERE  OrderDate >= '20130101' AND OrderDate < '20140101';
--   Index Seek, estimate from histogram, reads proportional to the year

-- Type mismatch
-- Column AccountNumber is varchar(20); parameter is nvarchar
SELECT * FROM Sales.Customer WHERE AccountNumber = N'AW00000042';
--   CONVERT_IMPLICIT(nvarchar, AccountNumber) -> scan (SQL collation); plan shows warning
```

Stale statistics case: after a nightly load of 2 million new rows, the histogram still ends at yesterday. A query for today's date estimates 1 row, picks nested loops with a key lookup, and executes 2 million lookups. `UPDATE STATISTICS Sales.SalesOrderHeader WITH FULLSCAN;` (or trace flag 2371 / compatibility ≥130's dynamic threshold) fixes it.

### 5. Under the hood
- **Update thresholds.** Classic rule: 500 + 20 % of rows. For compatibility level 130+ the threshold is dynamic: roughly `MIN(500 + 0.20·n, SQRT(1000·n))`, so a 10-million-row table needs about 100,000 changes, not 2 million.
- **Sampling.** Default sample size grows sub-linearly with table size; for skewed data a sample can miss rare but important values. Use `FULLSCAN`, `PERSIST_SAMPLE_PERCENT`, or filtered statistics for hot ranges.
- **Statistic is per leading column.** The histogram exists only for the first column; additional columns contribute through density. That's one reason for **multi-column statistics** or indexes to capture correlations (e.g. `City` and `ZipCode`).
- **Variables vs parameters.** A local variable's value is unknown at compile time, so the optimizer uses the density vector (average) rather than the histogram. A parameter's value is "sniffed" and uses the histogram.
- **PostgreSQL.** `ANALYZE` samples `300 × default_statistics_target` rows (default 100) and stores `n_distinct`, `most_common_vals`, `histogram_bounds`, `correlation` in `pg_stats`. `CREATE STATISTICS … (dependencies, ndistinct, mcv)` handles correlated columns. Autovacuum triggers `ANALYZE` after `autovacuum_analyze_threshold + scale_factor × rows` changes (default 50 + 10 %).
- **MySQL.** InnoDB persistent statistics from `ANALYZE TABLE`; histograms in 8.0 (`ANALYZE TABLE … UPDATE HISTOGRAM ON col`) help for non-indexed columns.
- **Why `LIKE '%abc'` can't seek.** Keys are ordered by prefix. Without a leading literal there's no contiguous range; use trigram (`pg_trgm`) or full-text indexes.

### 6. Performance and design implications
- Estimate accuracy matters more than raw index count; a wrong estimate misallocates memory, join algorithm and parallelism.
- Keep auto-create/auto-update on; consider async update for latency-sensitive OLTP and scheduled `FULLSCAN` for critical skewed columns.
- After large loads, partition switches or truncate/insert patterns, update statistics explicitly.
- Prefer persistent computed columns or expression indexes to make common transformations sargable.
- Check plans for **estimated vs actual rows** and for **warnings** (`CONVERT_IMPLICIT`, missing statistics). Extended Events `query_antipattern` and `plan_affecting_convert` help find offenders.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `DBCC SHOW_STATISTICS (tbl, stat)` | Header, density, histogram | Judge estimate quality |
| `UPDATE STATISTICS tbl [stat] WITH FULLSCAN` | Rebuild with all rows | After big loads / skew |
| `sys.dm_db_stats_properties()` | Last update, rows, modification counter | Monitoring staleness |
| `AUTO_UPDATE_STATISTICS_ASYNC` | Don't block queries to refresh | Latency-sensitive OLTP |
| `CREATE STATISTICS … WHERE …` | Filtered statistics | Hot subsets |
| `ANALYZE tbl` / `CREATE STATISTICS` (PostgreSQL) | Collect / extended stats | Correlated columns |
| `pg_stats`, `pg_stat_user_tables.last_autoanalyze` | Inspect stats | PostgreSQL diagnostics |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `FULLSCAN` on key skewed columns | Table is huge and window is small; use sampled with higher percent |
| Async auto-update | The first query after change must be accurate |
| Computed column / expression index | Column writes become hot and the computation expensive |
| Filtered statistics | Queries use parameters not matching the filter |
| Multi-column statistics | Many combinations; maintenance overhead |

### 9. Common mistakes
- `WHERE YEAR(col) = 2025`, `WHERE col + 1 = 5`, `WHERE ISNULL(col,0) = 0` → scans → move the work to the constant side or make it sargable.
- Mismatched types (`nvarchar` parameter vs `varchar` column) → implicit conversion → align types.
- Using local variables for filter values in a procedure → average-based estimates → use parameters or `OPTION (RECOMPILE)`.
- Assuming index rebuild updates all statistics → it updates that index's, with FULLSCAN, but not column statistics → update them separately (and `REORGANIZE` does not update at all).
- Ignoring ascending-key columns (dates, identities) → recent data estimated as 1 row → scheduled stats updates or trace-flag/CE improvements.

### 10. 🎯 Interview kit
**30-second answer:** Statistics summarise value distribution; the optimizer uses them to estimate row counts and choose seeks, scans and join methods. Sargable predicates let an index be navigated; functions or type conversions on the column prevent that, so I rewrite them and keep statistics fresh.

**Structure a longer answer:** estimation mechanism (histogram/density) → sargability rules → what goes wrong (stale, skew, correlation, ascending key) → detection (plan estimates vs actuals, stats DMVs) → fixes.

**Likely follow-ups:**
- *When is a statistic updated automatically?* → modification counter threshold, evaluated at compile time.
- *Why might a sargable query still scan?* → low selectivity or poor estimate.
- *Local variable vs parameter?* → density average vs histogram.
- *How do correlated columns hurt?* → independence assumption underestimates rows.

**Signals interviewers look for:** you link sargability to B-tree ordering and estimation to histograms, and you verify with actual vs estimated rows.

### 11. Check yourself
- [ ] Describe the columns of the SQL Server histogram output and how an equality estimate is read.
- [ ] Rewrite three non-sargable predicates.
- [ ] Explain the ascending-key problem and two mitigations.
- [ ] State the dynamic auto-update threshold idea.
- [ ] Give PostgreSQL's equivalent of `UPDATE STATISTICS` and of multi-column statistics.
<!-- explainer:end -->

## Practice questions

- [Basic] [What are table statistics and why does the query optimizer depend on them?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Execution_Plans/Q-25003_What_are_table_statistics_and_why_does_the_query_optimizer_d.md>)
- [Intermediate] [What does the ANALYZE command do in PostgreSQL and how is it different from VACUUM?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Execution_Plans/Q-25036_What_does_the_ANALYZE_command_do_in_PostgreSQL_and_how_is_it.md>)
- [Intermediate] [How does a DISTINCT clause typically appear in an execution plan and can it be expensive?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Execution_Plans/Q-25116_How_does_a_DISTINCT_clause_typically_appear_in_an_execution_.md>)
- [Intermediate] [How does the order of predicates in a WHERE clause affect the plan and does the optimizer care?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Execution_Plans/Q-25140_How_does_the_order_of_predicates_in_a_WHERE_clause_affect_th.md>)
- [Deep] [In an execution plan what is the difference between 'Rows Removed by Filter' and rows returned?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Execution_Plans/Q-25079_In_an_execution_plan_what_is_the_difference_between_Rows_Rem.md>)
- [Deep] [What is a histogram in database statistics and how does it help estimate skewed data distributions?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Execution_Plans/Q-25121_What_is_a_histogram_in_database_statistics_and_how_does_it_h.md>)
- [Deep] [How does table bloat from dead tuples affect execution plans and how do you detect and remediate it?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Execution_Plans/Q-24976_How_does_table_bloat_from_dead_tuples_affect_execution_plans.md>)
- [Deep] [When would you replace an expensive real-time aggregate query with a materialized view and how do you handle refresh?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Execution_Plans/Q-25028_When_would_you_replace_an_expensive_real-time_aggregate_quer.md>)

## Related concepts

- [Advanced Aggregation](<../../SQL_Language/Aggregation/Advanced_Aggregation.md>) *(Aggregation)*
- [Reading EXPLAIN](<../../SQL_Language/SQL_Fundamentals/Reading_EXPLAIN.md>) *(SQL Fundamentals)*
- [Partitioning and Partition Pruning](<../../Database_Internals/DBMS_Concepts/Partitioning_and_Partition_Pruning.md>) *(DBMS Concepts)*
- [Filtered Indexes and Plan Cache](<../../../TOOLS/SQL_Server/T-SQL/Filtered_Indexes_and_Plan_Cache.md>) *(T-SQL)*
- [Clustered Indexes and Indexed Views](<../../../TOOLS/SQL_Server/SQL_Server_Platform/Clustered_Indexes_and_Indexed_Views.md>) *(SQL Server Platform)*

## References

- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.)
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 2 Execution Plan Generation And The Query Optimizer
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 3 Methods For Capturing Query Performance Metrics
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Chapter 8. Queries, Modeling, and Transformation › The Query Optimizer
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › Use Execution Plans to Optimize Performance
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Reading Execution Plans
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Reading an Execution Plan Flow
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Estimated vs. Actual Execution Plans
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Infrastructure Operations › Essential database maintenance tasks
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — BatCh exeCution H XE UTION › Batch Mode Execution
- *Hadoop: The Definitive Guide* — Tom White — Crunch › Note

