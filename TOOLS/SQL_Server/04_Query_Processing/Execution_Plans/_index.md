---
id: "TOOLS/SQL_Server/04_Query_Processing/Execution_Plans"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Reading Execution Plans

**L4 Advanced** · [↑ Query Processing and Tuning](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** An execution plan is the tree of operators SQL Server runs to answer your query. Reading it means following the data flow from right to left, finding where estimated and actual row counts diverge, and checking the warnings and the operators that do the most work.

### 1. The big picture
The optimizer's decisions are invisible until you look at the plan. The plan shows which indexes are used and how (seek or scan), how tables are joined, where data is sorted, aggregated or spilled to tempdb, and how many rows the optimizer expected at each step. It is the main diagnostic tool for query tuning, and the rest of this block (plan cache, parameter sensitivity, Query Store) all store or compare plans.

A plan is **not** a performance measurement. It describes *behavior*. The cost percentages are estimates. To measure, you combine the plan with runtime metrics: actual rows, logical reads, CPU and elapsed time.

### 2. Core concepts and vocabulary
- **Operator**: one step in the plan (Index Seek, Clustered Index Scan, Nested Loops, Hash Match, Sort, Stream Aggregate, Key Lookup, Compute Scalar…). Each one has a *logical* name (what it does) and a *physical* name (how it does it).
- **Data flow arrows**: rows move from right to left. Arrow thickness shows the number of rows (estimated, or actual in an actual plan).
- **Estimated plan**: the plan produced without running the query (Ctrl+L, `SET SHOWPLAN_XML ON`, or the plan cache).
- **Actual plan ("plan with runtime metrics")**: the **same plan** plus what really happened: actual rows, executions, elapsed and CPU time per operator, memory used, spills. Collected with Ctrl+M or `SET STATISTICS XML ON`. For a given compiled plan, the operators are identical. Only the runtime numbers are added.
- **Estimated vs actual rows**: the most important comparison in a plan. A large gap (often 10× or more) means the cardinality estimate was wrong, and the plan was built on bad assumptions.
- **Number of executions**: how many times an operator ran. On the inner (bottom) side of a Nested Loops join, an operator runs once per outer row, so "Actual rows" there is the total across all executions.
- **Predicate vs Seek Predicate**: a *seek predicate* is used to navigate the B-tree. A *predicate* (residual) is checked on every row read. Large reads with few rows returned point to a residual predicate.
- **Warnings**: yellow triangles for implicit conversions that affect seeks, missing join predicates, spills to tempdb, excessive or insufficient memory grants, and missing statistics.
- **Missing index hint**: a green line suggesting an index. It is a hint, not a design. Evaluate it before creating anything.

### 3. How it works, step by step
**Capturing a plan:**
| Method | Runs the query? | Runtime metrics? |
|---|---|---|
| SSMS *Display Estimated Execution Plan* (Ctrl+L) / `SET SHOWPLAN_XML ON` | No | No |
| SSMS *Include Actual Execution Plan* (Ctrl+M) / `SET STATISTICS XML ON` | Yes | Yes |
| `sys.dm_exec_query_plan(plan_handle)` (plan cache) | Already ran | No (but DMV has aggregate stats) |
| `sys.dm_exec_query_statistics_xml(session_id)` | Running now | Partial, in flight (lightweight profiling, on by default since SQL Server 2019) |
| Query Store (`sys.query_store_plan`) | Already ran | Stored separately in runtime stats |
| Extended Events `query_post_execution_showplan` | Yes | Yes (expensive: use filters) |

**Reading a plan, in order:**
1. **Start at the root operator** (top left: SELECT, INSERT, UPDATE…). Open its *Properties* (F4). This node has the query-wide facts: Optimization Level, Reason For Early Termination, CE model version, compile time, memory grant, degree of parallelism, parameter compiled vs runtime values, and warnings.
2. **Read the warnings first.** They point at specific problems: implicit conversion, spill, missing statistics, missing join predicate.
3. **Follow the data from right to left.** The rightmost operators read data. Data flows left through joins, aggregates and sorts up to the root.
4. **Find the thick arrows and the high-cost operators.** These are where to look, but treat cost % as a pointer, not proof.
5. **Compare estimated vs actual rows at each operator**, starting at the right. The *first* (rightmost) place where they diverge widely is usually the root cause. Everything to its left inherits the error.
6. **Check how each table is accessed.** Seek or scan? Seek predicate or residual predicate? Key Lookups executed thousands of times?
7. **Check joins, sorts and aggregates.** Is the join type sensible for the actual row counts? Is a Sort there only because no index supplies the order?
8. **Look at the whole workload**, not one plan in isolation. A plan that is fine on its own can be the top CPU consumer because it runs 10,000 times a minute.

**The common operators:**
| Operator | Meaning | Look out for |
|---|---|---|
| Clustered Index Scan / Table Scan | Reads every row | Fine for big result sets; suspicious when few rows come out |
| Index Seek | Navigates the B-tree to a range | Check the seek predicate is what you expect |
| Key Lookup / RID Lookup | Fetches missing columns from the clustered index/heap per row | Many executions → consider a covering index |
| Nested Loops | For each outer row, probe the inner side | Great for small outer inputs; terrible when the outer estimate is too low |
| Merge Join | Merges two inputs sorted on the join key | Needs sorted input; watch for a Sort added just to feed it |
| Hash Match | Builds a hash table from one input, probes with the other | Needs a memory grant; spills to tempdb if under-granted |
| Sort | Orders rows | Memory and CPU; spills; often removable with an index |
| Stream / Hash Aggregate | GROUP BY / DISTINCT | Stream needs sorted input |
| Compute Scalar | Calculates expressions | Usually cheap; hides scalar UDF cost |
| Parallelism (Exchange) | Splits/merges row streams between threads | Skew between threads |
| Segment / Sequence Project | Window function bookkeeping | Harmless |

### 4. Worked example
```sql
SET STATISTICS IO, TIME ON;
-- Ctrl+M on
SELECT soh.SalesOrderID, soh.OrderDate, soh.TotalDue
FROM Sales.SalesOrderHeader AS soh
WHERE YEAR(soh.OrderDate) = 2013;
```
What the plan shows:
- An **Index Scan** (or clustered scan), not a seek, even if an index on `OrderDate` exists. `YEAR(OrderDate)` wraps the column in a function, so the optimizer cannot use the index order. The predicate is **non-SARGable**.
- The predicate appears as a residual *Predicate* on the scan. *Number of rows read* is the whole table, *actual rows* only a fraction.
- The estimate for a function on a column is a guess, so estimated and actual rows may differ.

Rewrite and compare:
```sql
SELECT soh.SalesOrderID, soh.OrderDate, soh.TotalDue
FROM Sales.SalesOrderHeader AS soh
WHERE soh.OrderDate >= '20130101' AND soh.OrderDate < '20140101';
```
With an index on `OrderDate`, you now see an **Index Seek** with a *Seek Predicate* on the range, a histogram-based estimate close to actual, and then either a Key Lookup for `TotalDue` (with many executions) or, if the range is large, a scan the optimizer still prefers. Compare logical reads in the Messages tab: that is your proof. If the lookups dominate, an index `(OrderDate) INCLUDE (TotalDue)` removes them.

**Reading a Nested Loops estimate trap:** if an inner-side seek shows *Estimated rows = 1* and *Actual rows = 50,000* with *Number of executions = 50,000*, the estimate is fine. Estimated rows on the inner side is **per execution**, actual is the **total**. Compare per execution before you blame statistics.

### 5. Under the hood
- A plan is stored as **XML** (showplan XML). The graphical view renders that XML. You can save it as `.sqlplan`, open it in SSMS or other tools, and query it with XQuery. All plan sources (cache, Query Store, Extended Events) store the same XML.
- **Execution model:** operators are *iterators*. Each one implements Open/GetNext/Close, and the root pulls rows from its children one row (or batch) at a time. This is why a `TOP (10)` can stop a scan early, and why *blocking* operators (Sort, Hash build, Hash Aggregate) must consume all their input before producing output and need memory grants.
- **Batch mode vs row mode:** with columnstore indexes (and batch mode on rowstore since SQL Server 2019, compatibility level 150+), operators process roughly 900 rows at a time. The operator property *Actual Execution Mode* shows Row or Batch.
- **Runtime metrics per operator** (actual elapsed and CPU time, logical reads per operator) are available in actual plans since SQL Server 2016 SP1/2017. Older versions only gave rows and executions.
- **Memory grant:** blocking operators need workspace memory reserved before execution, based on *estimated* rows. Too low → spill to tempdb (warning). Too high → wasted memory and queries waiting on `RESOURCE_SEMAPHORE`. Memory grant feedback (Intelligent QP) adjusts this between executions.

### 6. Performance and design implications
- Reading the actual plan is safe for SELECTs on test systems. For a data modification, wrap it in a transaction and roll back, or use the estimated plan.
- Capturing actual plans for every query (Extended Events `query_post_execution_showplan`, or trace) is expensive. Use Query Store and lightweight profiling in production.
- A big estimate gap is the most common root cause of bad plans. The usual causes are stale or missing statistics, skewed data, non-SARGable predicates (functions, implicit conversions, leading wildcards), table variables, multi-statement table-valued functions, and parameter sniffing.
- Don't create every missing-index suggestion: they ignore existing indexes, write cost and column order subtleties.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SET SHOWPLAN_XML ON` | Returns estimated plan XML, doesn't run | Production-safe look at the plan |
| `SET STATISTICS XML ON` | Runs and returns actual plan | Scripted actual plans |
| `SET STATISTICS IO, TIME ON` | Reads per table, CPU/elapsed per statement | Before/after numbers |
| `sys.dm_exec_query_plan(plan_handle)` | Cached plan XML | Plans for top queries from DMVs |
| `sys.dm_exec_text_query_plan(handle, start, end)` | Plan for one statement in a batch, as text | Large plans or statement-level |
| `sys.dm_exec_query_statistics_xml(spid)` | Live plan with in-flight row counts | Long-running query happening now |
| SSMS *Compare Showplan* | Highlights differences between two plans | Regressions, before/after |
| SSMS *Live Query Statistics* | Animated progress per operator | Watching long queries |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Estimated plan: query is long, modifies data, or production is under stress | Diagnosing estimate problems (no actual rows) |
| Actual plan: diagnosing; test environment | Data-modifying statements on production |
| Plan from cache / Query Store: need the plan production really used | It has no per-operator actual rows |
| Following cost % to find hot spots | Treating it as measured time |

### 9. Common mistakes
- Reading left to right → you start with the result, not the cause → start at the right, where data is read.
- Trusting cost % in an actual plan → the costs are still estimates even in the actual plan → use actual rows, reads and operator times.
- Comparing per-execution estimates with total actual rows on the inner side of loops → false "bad estimate" → divide by executions.
- Creating every missing-index suggestion → duplicate, overlapping indexes and slower writes → consolidate and test.
- Ignoring warnings on the root node → implicit conversions and spills are missed → always open the root properties first.

### 10. 🎯 Interview kit
**30-second answer:** "An execution plan is the operator tree the engine runs. An 'actual' plan is the same plan with runtime metrics added. I start at the root's properties and warnings, then read right to left, compare estimated vs actual rows to find where estimates first go wrong, check seeks vs scans, lookups, join types, sorts and spills, and confirm with logical reads and CPU rather than cost percentages."
**Structure a longer answer:** how to capture (SSMS, DMVs, Query Store) → reading order → estimate gaps and their causes → operator red flags → measure before/after.
**Likely follow-ups:** *Index seek vs scan, which is better?* → depends on rows needed; a scan returning most rows is fine. *What causes a spill?* → memory grant based on too-low estimates; fix estimates or rely on memory grant feedback. *What is a Key Lookup and how do you remove it?* → per-row fetch of missing columns; covering index with INCLUDE.
**Signals interviewers look for:** knows plans are estimates plus metrics; reads right to left; focuses on estimate vs actual; skeptical of cost % and missing-index hints.

### 11. Check yourself
- [ ] Capture the same query's plan from SSMS, the plan cache and Query Store, and confirm they match.
- [ ] Turn `YEAR(col) = 2013` into a SARGable range and show the change from scan to seek.
- [ ] Explain why actual rows on the inner side of Nested Loops can be far above the estimate without a bad estimate.
- [ ] Name four plan warnings and the cause of each.
- [ ] Explain why a Sort or Hash Match can spill to tempdb.
<!-- explainer:end -->

## In this section (study in order)

1. [Seeks Scans and Lookups](<Seeks_Scans_and_Lookups/_index.md>) (L5 Expert)
2. [Join Operators](<Join_Operators/_index.md>) (L5 Expert)
3. [Memory Grants and Spills](<Memory_Grants_and_Spills/_index.md>) (L5 Expert)

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Interpreting Operator Costs
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Using Actual Execution Plans for Real Metrics
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Estimated Execution Plans
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Introduction to Estimated vs. Actual Execution Plans
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Estimated vs. Actual Execution Plans
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Best Practices for Reading Execution Plans
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Comparing Estimated and Actual Plans
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Generating Execution Plans
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Estimated vs. Actual Execution Plans
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Understanding Estimated vs. Actual Execution Plans

