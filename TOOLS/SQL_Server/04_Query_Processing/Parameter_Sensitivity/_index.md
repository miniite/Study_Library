---
id: "TOOLS/SQL_Server/04_Query_Processing/Parameter_Sensitivity"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Parameter Sniffing and Sensitivity

**L4 Advanced** · [↑ Query Processing and Tuning](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** When a parameterized query is compiled, the optimizer "sniffs" the parameter values of that first call and builds a plan for them. If the data is skewed, that plan can be great for some values and terrible for others. This is the parameter-sensitive plan (PSP) problem.

### 1. The big picture
Parameter sniffing is not a bug. It is how plan reuse (previous lessons) and statistics (block 03) work together. When a procedure or `sp_executesql` statement compiles, the optimizer looks at the actual parameter values passed, reads the histogram for those values, and builds a plan for *that* row count. Then the plan is cached and reused for every later value.

- If values have similar row counts (uniform data), sniffing gives accurate plans: a win.
- If some values return 5 rows and others 500,000 (skewed data), the first caller decides the plan for everyone. If the "5 rows" plan (nested loops + key lookups) is cached, the 500,000-row call is very slow. If the "500,000 rows" plan (scan + hash join, large memory grant) is cached, the 5-row calls waste resources.

Classic symptoms: "the procedure was fine yesterday and is slow today, and nothing changed"; "it's fast when I run it in SSMS"; "restarting / `sp_recompile` fixes it for a while". Something caused a recompile (statistics update, plan eviction, failover), and the next caller happened to pass an unusual value.

### 2. Core concepts and vocabulary
- **Parameter sniffing**: using the parameter values present at compile time for cardinality estimation.
- **Parameter-sensitive plan (PSP)**: a query whose best plan depends on the parameter value.
- **Compiled value vs runtime value**: shown in the plan's root properties under *Parameter List* (`ParameterCompiledValue` and `ParameterRuntimeValue`). A big difference between their row counts is the fingerprint of a sniffing problem.
- **Local variable**: a `DECLARE @x` variable is **not** sniffed at normal compile time (its value isn't known yet), so the optimizer uses the **density vector** (average rows per value) instead of the histogram. During a statement-level recompile, variable values *are* known and are used.
- **`OPTIMIZE FOR (@p = value)`**: compile for a chosen value, whatever is passed.
- **`OPTIMIZE FOR UNKNOWN`**: ignore the sniffed value and use the average (density). Same effect as the local-variable trick.
- **`OPTION (RECOMPILE)`**: compile a plan for the actual values on every run.
- **PSP optimization (SQL Server 2022+, compatibility level 160)**: the optimizer can keep **several plan variants** for one parameterized statement, chosen at run time by how many rows the value is expected to return (low / medium / high cardinality buckets).

### 3. How it works, step by step
1. The first call `EXEC dbo.GetOrders @CustomerID = 1` arrives, and no plan is cached.
2. The optimizer looks up `1` in the histogram for `CustomerID`: 3 rows. It builds a plan for 3 rows (index seek + key lookup + nested loops, small memory grant).
3. The plan is cached with `ParameterCompiledValue = 1`.
4. The next call `@CustomerID = 11000` (400,000 rows) reuses the plan: 400,000 key lookups, nested loops running 400,000 times, sorts spilling because the grant was sized for 3 rows.
5. Later the statistics on the table are updated, the plan is invalidated, and the next caller's value determines the new plan. Performance flips back and forth depending on who calls first.

**How estimates differ by method:**
| Method | Row estimate comes from |
|---|---|
| Parameter (sniffed) | Histogram step for the compiled value |
| Local variable / `OPTIMIZE FOR UNKNOWN` | Density vector: total rows × density (an average) |
| `OPTIMIZE FOR (@p = 42)` | Histogram step for 42 |
| `OPTION (RECOMPILE)` | Histogram for the actual runtime value, every time |
| PSP optimization | Picks a variant by cardinality bucket of the runtime value |

### 4. Worked example
```sql
CREATE OR ALTER PROC dbo.ProductTransactionHistoryByReference
  (@ReferenceOrderID int)
AS
SELECT p.Name, p.ProductNumber, th.ReferenceOrderID
FROM Production.Product AS p
JOIN Production.TransactionHistory AS th ON th.ProductID = p.ProductID
WHERE th.ReferenceOrderID = @ReferenceOrderID;
GO
-- Value with many rows first
EXEC dbo.ProductTransactionHistoryByReference @ReferenceOrderID = 53465;
-- plan: scan of Product, merge join (+ sort); about 2 ms, ~245 reads

-- Remove ONLY this procedure's plan
DECLARE @ph varbinary(64);
SELECT @ph = plan_handle FROM sys.dm_exec_procedure_stats
WHERE object_id = OBJECT_ID(N'dbo.ProductTransactionHistoryByReference');
IF @ph IS NOT NULL DBCC FREEPROCCACHE(@ph);

-- Value with few rows first
EXEC dbo.ProductTransactionHistoryByReference @ReferenceOrderID = 816;
-- plan: seeks + nested loops; different shape
-- Now call it again with 53465: it reuses the nested-loops plan
```
How to read it:
- Open the root SELECT properties → *Parameter List*: compiled value 816, runtime value 53465.
- Compare estimated rows (based on 816) and actual rows (based on 53465) on the seek. The large gap is the evidence.
- Run both values with both plans and record reads and time. If the plans differ by milliseconds, live with sniffing. If one combination is catastrophic, choose a fix.

**Fix options applied to the statement:**
```sql
-- A: fresh plan every time (good for rarely-run queries)
… WHERE th.ReferenceOrderID = @ReferenceOrderID OPTION (RECOMPILE);

-- B: always plan for a typical value you choose
… OPTION (OPTIMIZE FOR (@ReferenceOrderID = 53465));

-- C: plan for the average (stable, never great for extremes)
… OPTION (OPTIMIZE FOR UNKNOWN);

-- D: without code changes, via Query Store (2022+)
EXEC sys.sp_query_store_set_hints @query_id = 42, @query_hints = N'OPTION (RECOMPILE)';
-- or force the plan that is acceptable for everyone
EXEC sys.sp_query_store_force_plan @query_id = 42, @plan_id = 7;
```

### 5. Under the hood
- **Why the local-variable trick works:** at batch compile time, a variable's value is unknown, so the optimizer uses `rows × all_density` for the column, the same estimate for every value. `OPTIMIZE FOR UNKNOWN` does the same thing without the code smell. Both give a *stable* plan, not an *optimal* one.
- **But variables are sniffed on recompile:** if the statement later recompiles at statement level, the variable now has a value and that value is used. So the local-variable trick is less "stable" than it looks.
- **PSP optimization (2022):** at compile time the optimizer finds parameters on predicates with skewed histograms. It creates a *dispatcher* plan and up to three *query variants*, one per cardinality range. At execution, the dispatcher evaluates the runtime value's estimated cardinality and runs the matching variant. Each variant is cached and tracked separately in Query Store (`sys.query_store_query_variant`). It works only for equality predicates and needs compatibility level 160. It reduces but does not remove the problem.
- **Related features:** memory grant feedback (2017+, percentile-based in 2022) reduces spills and over-grants when the grant swings between values. Adaptive joins (batch mode) choose loop or hash at run time. Both help with PSP symptoms without changing the plan shape problem.
- **SSMS vs application:** if you test with literals or local variables in SSMS, you get a different plan from the app's sniffed plan. Different SET options (`ARITHABORT`) also make SSMS compile its own plan. To reproduce, call the procedure with the app's parameter values and match the SET options, or get the cached plan.

### 6. Performance and design implications
- Not every plan change is a problem. Only fix when the *bad* combination costs real time at real frequency.
- Know your skew: check the histogram (`DBCC SHOW_STATISTICS`) for very large `EQ_ROWS` values next to tiny ones.
- Choose the fix by call frequency:
  - Called rarely, values vary wildly → `OPTION (RECOMPILE)`.
  - Called very often, one value dominates → `OPTIMIZE FOR (@p = typical)` or a forced plan.
  - Called often, plans differ little in cost → `OPTIMIZE FOR UNKNOWN`.
  - Two distinct populations (small and huge customers) → separate code paths (two procedures, or dynamic SQL so each gets its own plan) or PSP optimization on 2022+.
- "Catch-all" procedures (`WHERE (@a IS NULL OR col_a = @a) AND (@b IS NULL OR …)`) are a sniffing disaster. Use `OPTION (RECOMPILE)` or build dynamic SQL with `sp_executesql` that includes only the supplied predicates.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `OPTION (RECOMPILE)` | Plan per execution with real values | Rare or catch-all queries |
| `OPTION (OPTIMIZE FOR (@p = v))` | Plan for a fixed representative value | One dominant common case |
| `OPTION (OPTIMIZE FOR UNKNOWN)` | Plan for average density | Stable, "OK for all" plan |
| `ALTER DATABASE SCOPED CONFIGURATION SET PARAMETER_SNIFFING = OFF` | Disables sniffing for the whole database | Almost never; very broad |
| `ALTER DATABASE SCOPED CONFIGURATION SET PARAMETER_SENSITIVE_PLAN_OPTIMIZATION = ON` | PSP optimization (default ON at compat 160) | SQL Server 2022+ |
| `sys.sp_query_store_set_hints` | Attach hints without changing code | Third-party apps |
| `sys.sp_query_store_force_plan` | Force a known-good plan | Stable regression fix |
| `DBCC FREEPROCCACHE(plan_handle)` | Evict one plan | Emergency relief, not a fix |
| Plan root *Parameter List* | Compiled vs runtime values | Diagnosing |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `RECOMPILE`: low frequency, high variance | Hundreds of calls/sec, since compile CPU adds up |
| `OPTIMIZE FOR value`: a typical value covers most calls | That value's distribution changes over time |
| `OPTIMIZE FOR UNKNOWN`: need predictability | Extremes become consistently mediocre |
| Plan forcing: one plan is acceptable for all values | Data grows; review forced plans regularly |
| Code split / dynamic SQL: distinct populations | Code complexity, more plans to maintain |
| PSP optimization: SQL 2022+ equality predicates | Range predicates or multiple-parameter skew aren't covered |

### 9. Common mistakes
- "Fixing" sniffing by flushing the cache → works until the next unlucky compile → choose a real fix.
- Testing with literal values or local variables in SSMS → different plan from production, can't reproduce → use the procedure with real parameters and matching SET options.
- Copying parameters into local variables everywhere → average plans for all queries, and hidden recompile behavior → use `OPTIMIZE FOR UNKNOWN` only where needed.
- Turning off parameter sniffing database-wide → loses good plans for every well-behaved query → fix the few sensitive queries.
- Assuming every plan change is sniffing → missing stats or a schema change → check compiled vs runtime values first.

### 10. 🎯 Interview kit
**30-second answer:** "Parameter sniffing means the optimizer compiles a parameterized query using the parameter values of the first call and reuses that plan. With skewed data, a plan built for a rare value can be awful for a common one, or the reverse. I confirm it by comparing compiled and runtime parameter values and estimated vs actual rows in the plan, then choose a fix by call frequency: OPTION (RECOMPILE) for rare queries, OPTIMIZE FOR a typical value or forced plans via Query Store for hot ones, or splitting the code. On SQL Server 2022, PSP optimization can keep multiple plan variants."
**Structure a longer answer:** define → why it's usually good → when it goes wrong (skew) → how to prove it → fixes with trade-offs → scenario (procedure slow after a nightly stats update).
**Likely follow-ups:** *Why is it fast in SSMS but slow in the app?* → different SET options and values = different plan. *Why does a local variable change the plan?* → density vector instead of histogram. *What is PSP optimization?* → dispatcher plus variants by cardinality bucket, compat 160.
**Signals interviewers look for:** knows sniffing is normally beneficial; can prove it from the plan; picks fixes by frequency and skew; avoids blunt instruments (flushing the cache, disabling sniffing globally).

### 11. Check yourself
- [ ] Reproduce two different plans for one procedure by changing which value runs first.
- [ ] Find `ParameterCompiledValue` and `ParameterRuntimeValue` in a plan.
- [ ] Calculate the estimate a local variable gets from the density vector.
- [ ] Pick a fix for: (a) a monthly report, (b) an OLTP lookup called 2,000 times/sec, (c) a catch-all search procedure.
- [ ] Explain what PSP optimization does and its limits.
<!-- explainer:end -->

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Addressing Parameter Sniffing via Plan Reuse
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Parameter Sniffing and Plan Reuse
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Spotting Parameter Sniffing Issues
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Detecting Parameter Sniffing Issues
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 12 Dealing With Index Fragmentation
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Parameter Sniffing Awareness
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Handling Parameter Sniffing
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Advanced Hint Techniques
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Parameter Sniffing
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Statistics and Parameter Sniffing

