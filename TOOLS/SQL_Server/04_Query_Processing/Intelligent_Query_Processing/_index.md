---
id: "TOOLS/SQL_Server/04_Query_Processing/Intelligent_Query_Processing"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Intelligent Query Processing

**L4 Advanced** · [↑ Query Processing and Tuning](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Intelligent Query Processing (IQP) is a family of engine features, switched on by database compatibility level, that fix common optimizer weak spots automatically: adapting plans during execution, learning from previous executions (feedback), estimating better for functions and table variables, and running some aggregations approximately.

### 1. The big picture
The optimizer makes all its choices **before** execution, from estimates. When the estimates are wrong, the plan is wrong, and traditionally nothing corrects it until a human intervenes. Since SQL Server 2017, Microsoft has added features that close this gap:
- **Adapt during execution**: choose the join type after seeing the real row count (adaptive joins), or pause optimization to measure a function's output (interleaved execution).
- **Learn from the last execution** (*query processing feedback*): store corrections for memory grants, cardinality estimates, degree of parallelism and lock usage, and use them the next time.
- **Remove known blind spots**: get real row counts for table variables, inline scalar UDFs into the query, use batch mode on ordinary rowstore tables.
- **Trade precision for speed**: approximate distinct counts and percentiles.

Most features turn on **automatically when you raise the database compatibility level** (140 = 2017, 150 = 2019, 160 = 2022, 170 = 2025), with no code changes. That is the main reason to keep compatibility levels current, after testing with Query Store.

### 2. Core concepts and vocabulary
- **Compatibility level**: a per-database setting that controls optimizer and T-SQL behavior. IQP features are gated by it.
- **Database scoped configuration**: per-database switches to turn individual IQP features off (`ALTER DATABASE SCOPED CONFIGURATION SET … = OFF`).
- **`USE HINT ('DISABLE_…')`**: per-query switches to disable a feature.
- **Batch mode**: processing rows in groups (~900) with vectorized operators. Originally columnstore only, available on rowstore from 2019.
- **Memory grant**: workspace memory reserved before execution for sorts and hashes. Too low → spill to tempdb. Too high → wasted memory and waits.
- **Multi-statement table-valued function (MSTVF)**: a function that fills and returns a table variable. Historically a black box for the optimizer.
- **Scalar UDF**: a function returning one value, historically executed once per row and blocking parallelism.
- **Feedback persistence**: since 2022, feedback is stored in Query Store, so it survives plan eviction and restarts.

### 3. How it works, step by step
**The IQP feature map:**
| Feature | Compat | What it fixes | How |
|---|---|---|---|
| Interleaved execution for MSTVFs | 140 | Fixed guess of 1 row (old CE) or 100 rows (new CE) for MSTVFs | Pauses optimization, executes the MSTVF, uses its real row count, then finishes optimizing |
| Batch mode adaptive join | 140 (batch mode) | Wrong choice between nested loops and hash | Plan contains both; picks at run time once the build input row count passes or misses a threshold |
| Batch mode memory grant feedback | 140 | Spills / over-grants | Adjusts grant for next execution |
| Row mode memory grant feedback | 150 | Same, for row mode | Same |
| Percentile & persisted memory grant feedback | 160 | Grants oscillating between values; feedback lost on eviction | Uses a percentile of past usage; stores in Query Store |
| Table variable deferred compilation | 150 | Table variables estimated at 1 row | Compiles the statement on first use with the real row count |
| Batch mode on rowstore | 150 | Analytics on non-columnstore tables run in slow row mode | Uses batch operators when heuristics say it helps |
| Scalar UDF inlining | 150 | Per-row function calls, no parallelism | Rewrites the UDF body into the query as relational expressions |
| Approximate count distinct | any (2019+) | Exact `COUNT(DISTINCT)` on billions of rows | HyperLogLog sketch, ~2% error at 97% probability, little memory |
| `APPROX_PERCENTILE_CONT/DISC` | any (2022+) | Exact percentiles need big sorts | Sketch-based approximation |
| Cardinality estimation (CE) feedback | 160 | Repeated bad estimates from model assumptions | Tries a different CE assumption via hint, keeps it if better (stored in Query Store) |
| DOP feedback | 160 (config ON) | Too much parallelism wasting CPU | Gradually lowers DOP for a repeating query while elapsed time does not worsen |
| Parameter sensitive plan optimization | 160 | One plan for skewed parameter values | Multiple plan variants per cardinality range |
| Optimized plan forcing | 160 | Compile cost of forced plans | Stores compilation replay script |
| Optional parameter plan optimization (OPPO) | 170 | `col = @p OR @p IS NULL` patterns | Multiple variants depending on NULL vs value |
| CE feedback for expressions, optimized Halloween protection, more | 170 | Further estimate and plan fixes (SQL Server 2025) | Various |

**Memory grant feedback, in detail:**
1. A query runs. Its plan's grant was based on estimated rows.
2. At the end, the engine compares granted vs used memory. If granted memory is more than about 2× used (over-grant) or the query spilled (under-grant), it computes a better size.
3. It stores the new size with the cached plan (and from 2022 in Query Store). The Extended Event `memory_grant_updated_by_feedback` fires.
4. The next execution uses the adjusted grant. If the same value runs again, no further adjustment is needed. Small differences don't trigger readjustment.
5. If adjustments keep flipping (parameter-sensitive), the engine disables feedback for that plan. Percentile mode (2022) reduces this by aiming at a high percentile of recent usage.
6. The plan's root properties show `MemoryGrantInfo` → `IsMemoryGrantFeedbackAdjusted` (for example `YesAdjusting`, `YesStable`, `NoFirstExecution`, `NoFeedbackDisabled`) and `LastRequestedMemory`.

**Interleaved execution, in detail:**
1. During optimization, the optimizer finds a read-only MSTVF whose output feeds other operators.
2. It stops, executes the MSTVF subtree, and gets the real row count.
3. It resumes optimization with that number, so join types and memory grants fit. The plan is cached, so like parameter sniffing it is based on the first execution's values.

### 4. Worked example
**a) Nested MSTVFs (an anti-pattern IQP partly rescues)**
A common "object-oriented" style: `dbo.SalesInfo()` returns orders, `dbo.SalesDetails()` returns lines, and `dbo.CombinedSalesInfo()` joins those two functions, each one an MSTVF filling a table variable. Before SQL Server 2014 each function was estimated at **1 row**. With the new cardinality estimator (compat 120+) the guess is **100 rows**. Either way, the optimizer builds nested loops and small grants for tens of thousands of real rows.
```sql
ALTER DATABASE SCOPED CONFIGURATION SET INTERLEAVED_EXECUTION_TVF = OFF;
SELECT csi.OrderDate, csi.PurchaseOrderNumber, csi.OrderQty, sp.SalesQuota
FROM dbo.CombinedSalesInfo() AS csi
JOIN Sales.SalesPerson AS sp ON csi.SalesPersonID = sp.BusinessEntityID
WHERE csi.SalesPersonID = 277 AND csi.ShippingCity = 'Odessa';
ALTER DATABASE SCOPED CONFIGURATION SET INTERLEAVED_EXECUTION_TVF = ON;
-- run again and compare: estimated rows on the TVF now match actual
```
Even better, push the filters **into** a parameterized function (`dbo.AllSalesInfo(277, 'Odessa')`): estimates match exactly, and the plan switches from loops to hash. The improvement in the book's test was modest (about 8 ms → 6 ms), which is the real lesson: IQP **reduces** the damage of anti-patterns, but rewriting the code (inline table-valued functions or plain joins) is the real fix.

**b) Watching memory grant feedback**
```sql
CREATE EVENT SESSION MemoryGrant ON SERVER
ADD EVENT sqlserver.memory_grant_updated_by_feedback,
ADD EVENT sqlserver.memory_grant_feedback_loop_disabled
WITH (TRACK_CAUSALITY = ON);
ALTER EVENT SESSION MemoryGrant ON SERVER STATE = START;

EXEC dbo.CostCheck @Cost = 0;        -- grant 18,880 KB, used 1,440 KB → feedback event
EXEC dbo.CostCheck @Cost = 0;        -- no event: grant already adjusted
```
Check the root operator's `MemoryGrantInfo` in the actual plan to see the adjusted value.

**c) Approximate distinct count**
```sql
SELECT APPROX_COUNT_DISTINCT(CustomerID) AS approx_customers,
       COUNT(DISTINCT CustomerID)        AS exact_customers
FROM Sales.SalesOrderHeader;
```
On large tables the approximate version uses far less memory and avoids spills.

### 5. Under the hood
- **Adaptive join**: the plan contains an *Adaptive Join* operator with a hash-join branch and a nested-loops branch, plus an *adaptive threshold row count*. The build input is read into a buffer. If its row count reaches the threshold → hash. Otherwise → loops using the buffered rows. Properties show *Estimated/Actual Join Type*. It needs batch mode. On rowstore from 2019 it works via batch mode on rowstore.
- **Scalar UDF inlining** converts the function body (IF/ELSE, variable assignments, single RETURN) into relational operators (with `CASE`, `APPLY`), so the whole query is optimized together and can go parallel. Many constructs block inlining: `GETDATE()` and other time-dependent functions in some versions, table variables, loops, recursion, `EXECUTE AS`, and so on. Check `is_inlineable` in `sys.sql_modules`. Inlining can also produce large plans. Disable per function with `WITH INLINE = OFF`.
- **Feedback storage**: before 2022, feedback lived only with the cached plan and was lost on eviction or restart. From 2022, memory grant, CE and DOP feedback are stored in Query Store (`sys.query_store_plan_feedback`).
- **CE feedback** tries alternatives to model assumptions like *correlation* (independence vs partial correlation between predicates), *containment* for joins, and *row goals*. It applies a hint and keeps it only if the change helps.
- **DOP feedback** (off by default in 2022, config `DOP_FEEDBACK = ON`) lowers DOP in steps for queries running repeatedly in parallel with excessive waits, and stops when elapsed time gets worse.

### 6. Performance and design implications
- Raising compatibility level is the switch for most of these gains. Do it with Query Store on: capture at the old level, switch, compare, force plans where needed.
- IQP features reduce symptoms of bad estimates, but bad patterns (nested MSTVFs, scalar UDFs, table variables with large data, catch-all queries) are still worse than good code. Prefer inline TVFs, set-based logic and temp tables for large data.
- Because several features cache decisions based on the first execution (interleaved execution, deferred compilation of table variables), they can behave like parameter sniffing.
- If one feature causes a regression, disable it for that query (`USE HINT`) or that database (scoped configuration) rather than lowering the whole compatibility level.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ALTER DATABASE … SET COMPATIBILITY_LEVEL = 160` | Enables the level's IQP features | Planned upgrade with Query Store |
| `ALTER DATABASE SCOPED CONFIGURATION SET INTERLEAVED_EXECUTION_TVF = OFF` | Disable one feature per database | A specific regression |
| Similar: `BATCH_MODE_ADAPTIVE_JOINS`, `ROW_MODE_MEMORY_GRANT_FEEDBACK`, `TSQL_SCALAR_UDF_INLINING`, `DEFERRED_COMPILATION_TV`, `BATCH_MODE_ON_ROWSTORE`, `CE_FEEDBACK`, `DOP_FEEDBACK`, `MEMORY_GRANT_FEEDBACK_PERCENTILE_GRANT`, `PARAMETER_SENSITIVE_PLAN_OPTIMIZATION` | Per-feature switches | Same |
| `OPTION (USE HINT ('DISABLE_BATCH_MODE_ADAPTIVE_JOINS'))` | Disable per query | Narrow fix |
| `CREATE FUNCTION … WITH INLINE = OFF` | Prevent UDF inlining | Inlined plan is worse |
| `sys.sql_modules.is_inlineable` | Can the UDF be inlined? | Checking UDFs |
| `APPROX_COUNT_DISTINCT(col)` | Approximate distinct count | Large analytics where ±2% is fine |
| XE `memory_grant_updated_by_feedback` | Shows feedback adjustments | Verifying feedback |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Raise compat level to get IQP | Without a Query Store baseline and testing |
| Rely on memory grant feedback | Parameter-sensitive queries where grants flip; feedback disables itself |
| Scalar UDF inlining | Very large UDFs producing huge plans; disable per function |
| Approximate functions: dashboards, exploration | Billing, finance, anything needing exact counts |
| Interleaved execution helps MSTVFs | As an excuse to keep nested MSTVFs instead of inline TVFs |

### 9. Common mistakes
- Staying on an old compatibility level "for safety" forever → missing years of optimizer fixes → upgrade with Query Store and fix the few regressions.
- Lowering the compatibility level to fix one regression → lose all features for all queries → disable the one feature or use a Query Store hint.
- Assuming IQP fixes bad code → nested MSTVFs and RBAR remain slow → rewrite.
- Expecting feedback on the first execution → it only helps the *next* execution → judge after a few runs.
- Using approximate functions where exact numbers are required → wrong totals → use only where small error is acceptable.

### 10. 🎯 Interview kit
**30-second answer:** "Intelligent Query Processing is a set of features enabled by compatibility level that compensate for bad estimates. Adaptive joins choose hash or loops at run time; interleaved execution measures multi-statement TVF output before finishing optimization; memory grant, CE and DOP feedback learn from previous executions and, since 2022, persist in Query Store; table variable deferred compilation and scalar UDF inlining remove old blind spots; and there are approximate aggregates. I raise compatibility levels with a Query Store baseline and disable individual features only for specific regressions."
**Structure a longer answer:** why (estimates fixed before execution) → three families (adaptive, feedback, blind-spot removal) plus approximate → enablement and switches → limits (doesn't fix anti-patterns, sniffing-like behavior).
**Likely follow-ups:** *What does memory grant feedback do?* → adjusts grant after over/under-use; percentile and persisted in 2022. *How does an adaptive join decide?* → threshold row count on the build input. *How do you disable one feature?* → database scoped configuration or `USE HINT`.
**Signals interviewers look for:** ties features to compat levels; knows they reduce but don't remove bad-code costs; can switch features off surgically.

### 11. Check yourself
- [ ] Show an MSTVF's estimate with interleaved execution off and on.
- [ ] Trigger and capture `memory_grant_updated_by_feedback`, then read `MemoryGrantInfo`.
- [ ] Check whether your scalar UDFs are inlineable.
- [ ] Explain the threshold mechanism of an adaptive join.
- [ ] Plan a compatibility-level upgrade from 130 to 160 using Query Store.
<!-- explainer:end -->

## References

- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.)
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 21 Intelligent Query Processing
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Conclusion
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › Introduction to Advanced SQL Features
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 15: Case Studies and the Future of T-SQL › Emerging AI-Assisted Development
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › Best Practices for Joins, Subqueries, and CTEs
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Combining Query Rewrites with Hints
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 15: Case Studies and the Future of T-SQL › Conclusion
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Performance Considerations
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 15: Case Studies and the Future of T-SQL › Query Refactoring

