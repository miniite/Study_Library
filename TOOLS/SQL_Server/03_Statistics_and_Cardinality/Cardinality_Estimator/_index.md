---
id: "TOOLS/SQL_Server/03_Statistics_and_Cardinality/Cardinality_Estimator"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Cardinality Estimator

**L4 Advanced** · [↑ Statistics and Cardinality](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** The cardinality estimator is the model that combines statistics into row estimates for every operator; SQL Server has a legacy model (CE 70) and a modern one (CE 120+), which make different assumptions about correlated predicates, joins and new data, and you choose between them by compatibility level or hints.

### 1. The big picture
Statistics give the optimizer facts about single columns: a histogram here, a density there. But real queries have several predicates (`Color = 'Red' AND DaysToManufacture > 15`), joins, `GROUP BY`s, functions and variables. Something must decide how to *combine* those facts. That is the **cardinality estimator (CE)**. It is a set of rules and assumptions built into the optimizer.

In SQL Server 2014 Microsoft rewrote the CE. The old one (now called *legacy*, model version 70) dates from SQL Server 7.0. The new one is versioned with the compatibility level (120 for 2014, 130 for 2016 … 160 for 2022, 170 for 2025). Most workloads got better plans with the new CE, but some got worse, so you need to know how to tell which model a plan used, how to switch, and how to diagnose bad estimates in general.

### 2. Core concepts and vocabulary
- **Cardinality estimate**: predicted row count for an operator's output.
- **Selectivity**: fraction of input rows a predicate keeps.
- **CE model version**: `CardinalityEstimationModelVersion` property on the plan's root operator: 70 = legacy, 120+ = modern.
- **Independence assumption** (legacy): predicates on different columns are unrelated, so selectivities multiply.
- **Exponential backoff** (modern): assumes some correlation; multiplies the most selective predicate fully and the next ones only partly.
- **Simple containment** (legacy join assumption): rows that pass filters on each side still join as if the filters were not there.
- **Base containment** (modern join assumption): estimates the join from the base tables' histograms, then applies filters.
- **Ascending key**: a leading column whose new values are always beyond the histogram's last step.
- **Fixed guesses ("magic numbers")**: default selectivities when no statistic can help, e.g. 30 % for an unknown inequality.
- **Underestimate / overestimate**: actual > estimate / actual < estimate.

### 3. How it works, step by step
**3a. Single predicate.** Use the histogram (known value) or density (unknown value), as in *Histogram and Density*.

**3b. Several predicates on one table (AND).** Suppose selectivities are 0.1 (`Color = 'Red'`) and 0.05 (`DaysToManufacture > 15`) on 1,000 rows.

| Model | Formula | Estimate |
|---|---|---|
| Legacy (independence) | 0.05 × 0.1 = 0.005 | 5 rows |
| Modern (exponential backoff) | s1 × s2^(1/2) × s3^(1/4) × s4^(1/8), most selective first: 0.05 × 0.1^(1/2) ≈ 0.0158 | ≈ 16 rows |

Real data is often correlated (red products may be exactly the slow-to-make ones), so the legacy model tends to underestimate. Backoff gives a middle ground. If a multi-column statistic exists for an equality pair, both models can use the combined density instead.

**3c. OR predicates.** Selectivities are combined with inclusion–exclusion (s1 + s2 − s1×s2) with similar backoff adjustments in the modern CE.

**3d. Joins.** For `A JOIN B ON A.k = B.k` the estimator aligns the two histograms on the join column.
- **Legacy** (simple containment): estimate the join with step-by-step histogram alignment, then assume filters on either side do not change which values match.
- **Modern** (base containment): estimate the join from the base tables' histograms at a coarser level, then scale by each side's filter selectivity. It assumes filter and join columns are independent.
Neither is always right; they fail in different situations.

**3e. Values beyond the histogram (ascending keys).**
- **Legacy**: a value above the last `RANGE_HI_KEY` is estimated at about 1 row. A query for "today's orders" right after a load gets 1 row, a loop-join plan, and runs slowly.
- **Modern**: uses the column's average frequency (density × rows) for out-of-range values, usually much closer.

**3f. Things the CE can't see.**
- **Non-SARGable expressions** (`WHERE YEAR(OrderDate) = 2024`, `WHERE UPPER(Name) = 'X'`): no histogram applies; fixed guesses are used.
- **Local variables**: density, not histogram.
- **Table variables**: 1 row before SQL Server 2019; from 2019 (compat 150) *deferred compilation* uses the real count at first execution.
- **Multi-statement table-valued functions**: legacy 1 row, modern 100 rows; *interleaved execution* (2017+) uses the real count.
- **Unknown inequality**: 30 % of rows. Combined fixed guesses compound quickly.

**3g. Where estimates go wrong and why it matters.**
- **Underestimate** → small memory grant → Sort/Hash spills to tempdb (warning icons); nested loops over large inputs; serial plan.
- **Overestimate** → huge memory grant, `RESOURCE_SEMAPHORE` waits for other sessions, needless parallelism and scans.
- Errors **compound** through a plan: an error at the bottom multiplies at every join above it.

### 4. Worked example
```sql
-- 1) Which CE did this plan use? Look at the root operator's properties:
--    CardinalityEstimationModelVersion = 70 (legacy) or 120..170 (modern).

-- 2) Compare the two models on the same query
SELECT p.Name, p.Class
FROM Production.Product p
WHERE p.Color = 'Red' AND p.DaysToManufacture > 15;                       -- default (by compat level)

SELECT p.Name, p.Class
FROM Production.Product p
WHERE p.Color = 'Red' AND p.DaysToManufacture > 15
OPTION (USE HINT ('FORCE_LEGACY_CARDINALITY_ESTIMATION'));                -- legacy

-- 3) Database-wide switch (affects every query in the database)
ALTER DATABASE SCOPED CONFIGURATION SET LEGACY_CARDINALITY_ESTIMATION = ON;
ALTER DATABASE SCOPED CONFIGURATION SET LEGACY_CARDINALITY_ESTIMATION = OFF;

-- 4) Find operators with big misestimates in a captured actual plan:
--    compare EstimateRows vs ActualRows (per operator, per thread) in the plan XML,
--    or use Query Store / sys.dm_exec_query_stats (last_rows vs plan estimates).

-- 5) Watch estimates being made (Debug channel: test systems only, short sessions)
CREATE EVENT SESSION CardinalityEstimation ON SERVER
ADD EVENT sqlserver.auto_stats (WHERE sqlserver.database_name = N'AdventureWorks'),
ADD EVENT sqlserver.query_optimizer_estimate_cardinality
    (WHERE sqlserver.database_name = N'AdventureWorks')
ADD TARGET package0.event_file (SET filename = N'cardinalityestimation')
WITH (TRACK_CAUSALITY = ON);
```
**How to read it:** In query 2, compare *Estimated Number of Rows* on the filter/scan operator between the two plans: the legacy plan should show a smaller estimate (independence), the modern one a larger one (backoff). The XE event `query_optimizer_estimate_cardinality` shows an `input_relation` (what was being estimated) and `stats_collected` (which statistics were used) for each estimate, which tells you *where* a number came from.

### 5. Under the hood
- **The CE is versioned with compatibility level.** Upgrading SQL Server does not change the CE until you raise the database compatibility level. That is the main reason to test a compat-level change with Query Store capturing plans before and after.
- **Ways to pick the CE, from broad to narrow:**
  1. Database compatibility level (110 or lower → legacy; 120+ → modern of that version).
  2. `ALTER DATABASE SCOPED CONFIGURATION SET LEGACY_CARDINALITY_ESTIMATION = ON` (keeps other new features of the compat level).
  3. Trace flag 9481 (legacy) / 2312 (modern): old-fashioned, server or query level; avoid when a scoped config or hint exists.
  4. Query hint `USE HINT ('FORCE_LEGACY_CARDINALITY_ESTIMATION')` or `'FORCE_DEFAULT_CARDINALITY_ESTIMATION'`.
  5. Query Store hints (SQL Server 2022+, Azure SQL): attach the hint to a query without changing code.
- **Other model hints:** `ASSUME_JOIN_PREDICATE_DEPENDS_ON_FILTERS` (simple containment under the modern CE), `ASSUME_MIN_SELECTIVITY_FOR_FILTER_ESTIMATES` (independence under the modern CE), `ENABLE_HIST_AMENDMENT_FOR_ASC_KEYS`.
- **CE feedback (SQL Server 2022, compat 160)**: with Query Store on, the engine can notice repeated misestimates caused by a model assumption and automatically apply a different assumption (e.g. drop backoff) for that query.
- **Setting compat level to 110 just to get the legacy CE** also removes many newer optimizer features. Prefer the scoped configuration or a targeted hint.

### 6. Performance and design implications
- **First diagnose, then choose a model.** Most misestimates come from stale stats, non-SARGable predicates, local variables, table variables or skew, not from the CE version.
- **Test compat-level upgrades** with Query Store: capture a baseline, raise the level, compare regressions, force good plans or add hints where needed.
- **Help the estimator:** multi-column statistics for correlated predicates, computed columns (with statistics) for expressions, temp tables instead of table variables when row counts matter, `RECOMPILE` for highly variable queries.
- **Measure:** actual plan (estimated vs actual rows per operator), spill warnings, memory grant info (`sys.dm_exec_query_memory_grants`), Query Store runtime stats.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ALTER DATABASE ... SET COMPATIBILITY_LEVEL = 160` | Selects optimizer/CE version | Upgrades (test first) |
| `ALTER DATABASE SCOPED CONFIGURATION SET LEGACY_CARDINALITY_ESTIMATION = ON` | Legacy CE for the database | Many regressions after upgrade |
| `OPTION (USE HINT ('FORCE_LEGACY_CARDINALITY_ESTIMATION'))` | Legacy CE for one query | A single regressed query |
| `OPTION (USE HINT ('ASSUME_MIN_SELECTIVITY_FOR_FILTER_ESTIMATES'))` | Independence for filters | Truly uncorrelated predicates |
| `sys.sp_query_store_set_hints` | Attach hints without code change (2022+) | Vendor code |
| XE `query_optimizer_estimate_cardinality` | Shows each estimate and its inputs | Deep debugging, test only |
| XE `inaccurate_cardinality_estimate` | Fires on big misestimates (debug) | Short, filtered sessions |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Modern CE (default) for new and upgraded databases | Specific queries regress; fix those, don't revert the whole DB |
| Legacy CE via scoped config after a broad regression | Long term; you miss new improvements |
| Per-query hints / Query Store hints | Many queries need them (look for a root cause) |
| `RECOMPILE` to get exact values each time | Very frequent queries (compile CPU) |

### 9. Common mistakes
- Lowering compat level to 110 to get the legacy CE → loses newer features → use `LEGACY_CARDINALITY_ESTIMATION` instead.
- Upgrading compat level without a Query Store baseline → can't prove what regressed → capture first.
- Blaming the CE for a non-SARGable predicate → fixed guess either way → rewrite the predicate.
- Leaving debug XE events running in production → overhead and noise → short, filtered sessions only.
- Using table variables for thousands of rows on old versions → 1-row estimate → temp tables or compat 150+.

### 10. 🎯 Interview kit
**30-second answer:** The cardinality estimator combines statistics into row estimates. The legacy CE assumes predicates are independent (multiply selectivities), uses simple containment for joins and estimates 1 row beyond the histogram. The modern CE (2014+, tied to compat level) uses exponential backoff for correlated predicates, base containment for joins and average frequency for out-of-range values. You see which one ran in `CardinalityEstimationModelVersion` and can switch per database or per query.

**Structure a longer answer:** clarify → assumptions of each model with a numeric example → how to tell and switch → diagnosing misestimates (estimated vs actual, spills, grants) → upgrade story with Query Store.

**Likely follow-ups:**
- *After a compat-level upgrade some queries got slower. What do you do?* → Query Store compare, force last good plan, per-query hint, root-cause.
- *What's exponential backoff?* → most selective predicate full, then square root, fourth root…
- *Why do table variables estimate badly?* → 1 row before 2019; deferred compilation since.

**Signals interviewers look for:** concrete assumptions, not just "the new CE is better"; knows the scoped configuration and hints; diagnoses before switching models.

### 11. Check yourself
- [ ] Compute legacy and modern estimates for three predicates with selectivities 0.2, 0.1, 0.5 on 100,000 rows.
- [ ] Find `CardinalityEstimationModelVersion` in a plan.
- [ ] Explain the ascending-key problem and how each CE handles it.
- [ ] Describe a safe plan for raising compatibility level on a production database.
<!-- explainer:end -->

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Using Actual vs. Estimated Metrics for Operators
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Looking at Estimated vs. Actual Rows
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 5 Statistics, Data Distribution, And Cardinality
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Identifying Cardinality Estimation Issues
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Common Pitfalls in Estimated Plans
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Population

