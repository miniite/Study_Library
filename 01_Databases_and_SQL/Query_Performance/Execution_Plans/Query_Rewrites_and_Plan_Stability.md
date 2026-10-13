---
id: "01_Databases_and_SQL/Query_Performance/Execution_Plans/Query_Rewrites_and_Plan_Stability"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Query_Performance/Execution_Plans"
tools: []
---

# Query Rewrites and Plan Stability

**L4 Advanced** · [↑ Execution Plans](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Most slow queries are fixed by rewriting them so the optimizer can use indexes and good estimates (sargable predicates, fewer IN/OR terms, no row-by-row logic), and plan stability means keeping a good plan from flipping when parameters, statistics or versions change.

### 1. The big picture
The optimizer does cost-based search over the plans that are *possible* for the query as written. A rewrite changes the space of possible plans or the quality of the estimates the optimizer sees. Two broad families of rewrite cover most cases:

1. **Make the predicate searchable** — so an index seek is even possible (sargability).
2. **Make the work smaller or simpler** — avoid huge `IN` lists and many `OR`s, avoid `SELECT *`, replace scalar user-defined functions and cursors with set-based logic, and move filters before joins/aggregations where the engine cannot.

Separate from rewriting is **plan stability**: given a good query, how do you make sure it keeps getting a good plan? A plan is chosen at compile time using the parameter values and statistics visible *then*. If those change, the plan can flip to a worse one. Stability techniques range from sensible habits (up-to-date statistics, parameterised queries) to heavy hammers (hints, plan guides, forced plans).

### 2. Core concepts and vocabulary
- **Sargable (Search ARGument-able)** — a predicate that an index seek can use because the indexed column is compared bare to a value.
- **Non-sargable** — column wrapped in a function/expression, leading-wildcard `LIKE`, implicit conversion on the column side.
- **Implicit conversion** — the engine converts one side's type to match the other; if it converts the *column*, the index cannot seek.
- **Index seek vs scan** — navigate the B-tree to a range vs read everything.
- **Covering index / key lookup** — an index containing all needed columns vs having to fetch the rest from the base row.
- **Parameter sniffing** — compile for first-seen parameter values.
- **Plan cache** — memory structure holding compiled plans for reuse.
- **Hint** — instruction forcing an index, join type, or recompile behaviour.
- **Plan guide / Query Store hint / forced plan** — ways to attach hints or a plan to a query without changing its text.
- **Anti-pattern** — a known problematic query shape (SQL Server 2022's `query_antipattern` extended event flags several).

### 3. How it works, step by step
**Rewrite loop**
1. Get the actual plan and note: scans on big tables, key lookups executed many times, estimate-vs-actual gaps, warnings (implicit conversion, spills).
2. Find the predicate/expression that prevents a seek or misleads estimates.
3. Rewrite preserving semantics.
4. Re-run and compare `STATISTICS IO` reads and time, plus result equality (row counts / checksum).
5. Add or adjust an index only if the rewrite alone doesn't suffice.

**Common rewrites**

| Before | After | Why |
|---|---|---|
| `WHERE YEAR(OrderDate) = 2025` | `WHERE OrderDate >= '20250101' AND OrderDate < '20260101'` | Function on column blocks seek |
| `WHERE CHARINDEX('Bear', Name) = 1` | `WHERE Name LIKE 'Bear%'` | Prefix `LIKE` is a range seek |
| `WHERE Amount * 1.2 > 100` | `WHERE Amount > 100/1.2` | Move arithmetic off the column |
| `WHERE CustomerID = '123'` (int column vs varchar literal is fine; varchar column vs int/nvarchar param is not) | match parameter type to the column | Avoid conversion on column |
| `IN (… 5000 literals …)` | join to a temp table / table-valued parameter | Better stats, smaller plan |
| `WHERE a=1 OR b=2` | `UNION ALL` of two seekable queries (with a guard to avoid dupes) | One seek per index |
| `SELECT *` | list needed columns | Enables covering index |
| Scalar UDF per row / cursor | inline table-valued function / set-based `JOIN` | UDFs and cursors serialize and run row by row |
| `WHERE NOT IN (subquery)` | `NOT EXISTS` | NULL semantics, better anti-join |

### 4. Worked example
```sql
-- Slow: function on column + leading predicate
SELECT SalesOrderID, OrderDate
FROM   Sales.SalesOrderHeader
WHERE  DATEPART(YEAR, OrderDate) = 2025 AND DATEPART(MONTH, OrderDate) = 3;
-- Plan: Index Scan of ~30k pages; estimate far off (guess 10% of table)

-- Rewrite: half-open date range
SELECT SalesOrderID, OrderDate
FROM   Sales.SalesOrderHeader
WHERE  OrderDate >= '2025-03-01' AND OrderDate < '2025-04-01';
-- Plan: Index Seek on IX_OrderDate, reads ~40 pages
```

The rewrite is not just faster; the optimizer can now use the **histogram** on `OrderDate` to estimate rows correctly, which also improves the join method chosen above this operator.

Plan stability for a parameterised search:

```sql
CREATE OR ALTER PROCEDURE dbo.usp_Search @Status tinyint AS
SELECT OrderID, CustomerID FROM dbo.Orders WHERE Status = @Status;
-- Status = 9 ('archived') is 95% of the rows; Status = 1 is 0.1%.
-- Cached plan from the first call decides everything.
-- Options:
--   WITH RECOMPILE (whole proc) or OPTION (RECOMPILE) per statement
--   OPTION (OPTIMIZE FOR (@Status = 1))  -- bias to the common/OLTP case
--   OPTION (OPTIMIZE FOR UNKNOWN)        -- average-based, "mediocre for all"
--   IF @Status = 9 EXEC dbo.usp_SearchArchive ELSE EXEC dbo.usp_SearchLive;
```

### 5. Under the hood
- **Why functions break seeks.** B-tree keys are stored in the order of the raw column value. `f(col)` has no stored order, so the engine must evaluate `f` for every row. Exceptions: SQL Server can rewrite some conversions (`CAST(datetime AS date)` is sargable in modern versions), and PostgreSQL supports **expression indexes** (`CREATE INDEX ON t (lower(email))`), SQL Server uses **computed columns + index**.
- **Large IN lists / many ORs.** There's no fixed limit, but past ~100 values, and certainly past 1,000, compile time and plan quality suffer: each value is a separate seek or a constant scan; estimation uses a combined selectivity formula that is often wrong. Table-valued parameters or temp tables give the optimizer a real object with statistics.
- **Plan caching and sniffing.** On first execution the optimizer reads the parameter's actual value, estimates from the histogram, caches the plan, and reuses it. A plan good for 3 rows may be disastrous for 3 million. SQL Server 2022 *Parameter Sensitive Plan (PSP)* can compile up to three plans per statement (small/medium/large) selected by a dispatcher on the parameter value.
- **PostgreSQL.** Prepared statements use *custom* plans for the first five executions, then compare the average cost to a *generic* plan; `plan_cache_mode = force_custom_plan` avoids generic-plan skew problems.
- **Query Store hints** (SQL Server 2022) attach `OPTION` hints to a query without code changes; **plan guides** do the same by text matching and are fragile.
- **Why statistics are part of stability.** Auto-update triggers after roughly `sqrt(1000 × rows)` modifications (modern thresholds), invalidating dependent plans; a nightly large load can therefore create a new plan on first call the next morning, from an unrepresentative parameter.

### 6. Performance and design implications
- Always measure **logical reads** and **plan operators**, not only duration.
- Prefer set-based statements; each row-by-row loop multiplies round trips, log writes and plan overheads.
- Keep types aligned between application parameters, variables and columns (use `varchar` vs `nvarchar` consistently; ORMs often send `nvarchar`).
- ORMs: watch for `IN` lists, `OR`-chains built dynamically, and `SELECT *`; use batching or TVPs.
- Be conservative with hints: a hint is a promise that the data shape will never change.
- Parameterize ad hoc SQL (or enable forced parameterization carefully) so plans are reused — but handle skew explicitly.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `OPTION (RECOMPILE)` | Fresh plan with actual values each run | Skewed parameters, infrequent heavy queries |
| `OPTION (OPTIMIZE FOR (@p = v | UNKNOWN))` | Choose compile-time value | Known typical case |
| `sp_query_store_set_query_hints` | Attach hints without code change | Third-party / ORM SQL |
| `sp_query_store_force_plan` | Pin a plan | Stabilise a known-good plan |
| `ALTER DATABASE SCOPED CONFIGURATION SET PARAMETER_SNIFFING = OFF` | Disable sniffing for the DB | Last resort, test first |
| `CREATE INDEX … ON t (lower(col))` (PostgreSQL) | Expression index | Case-insensitive lookups |
| `plan_cache_mode` (PostgreSQL) | Generic vs custom plans | Prepared-statement skew |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Rewrite predicates to be sargable | The "rewrite" changes semantics (NULLs, time zones, end-of-day) |
| `RECOMPILE` | Query runs at high frequency (compile CPU) |
| `OPTIMIZE FOR` a value | Distribution shifts and the chosen value stops being typical |
| Forced plan | Underlying index may be dropped or data profile changes |
| Dynamic SQL per variant | You can't guard against injection or plan-cache bloat |

### 9. Common mistakes
- `WHERE CONVERT(varchar, col) = @x` → scan → compare using the column's own type.
- Using `BETWEEN` with datetimes and `23:59:59` → misses rows → use `>= start AND < next_start`.
- Adding indexes for every slow query → write overhead → rewrite first, index second.
- `OPTION (FORCE ORDER)` or index hints scattered in code → brittle → prefer fixing statistics and structure.
- Rebuilding statistics with a tiny sample on a skewed column → poor estimates → increase sample.
- Believing the plan will "always be the same" after deployment → test with realistic parameter sets (small, large, NULL, missing).

### 10. 🎯 Interview kit
**30-second answer:** First make the predicates sargable and the query set-based so the optimizer can seek and estimate well; then, for stability, keep statistics current and handle parameter skew with recompile, optimize-for, PSP or Query Store forcing — with monitoring.

**Structure a longer answer:** read the plan → identify non-sargable/oversized constructs → rewrite → verify reads/time → decide stability approach by frequency and skew → monitor.

**Likely follow-ups:**
- *Why does a function on a column cause a scan?* → no usable ordering; fix by rewrite, expression index or computed column.
- *`IN` vs `EXISTS` vs `JOIN`?* → semantics with NULLs and duplicates; optimizer often equal for positive forms; use `NOT EXISTS` for anti-joins.
- *What is parameter sniffing and three fixes?*
- *When is a plan guide better than code change?* → vendor/ORM code you cannot edit.

**Signals interviewers look for:** you rewrite with measurements, understand estimation, and treat hints as a trade-off.

### 11. Check yourself
- [ ] Rewrite `WHERE MONTH(d)=3 AND YEAR(d)=2025` sargably.
- [ ] Explain why a 5,000-literal `IN` is a problem and the alternative.
- [ ] Describe parameter sniffing and four mitigation options.
- [ ] Say how PostgreSQL decides between generic and custom plans.
- [ ] List risks of forcing a plan.
<!-- explainer:end -->

## Practice questions

- [Basic] [How does creating an index on a WHERE-clause column typically change the execution plan?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Execution_Plans/Q-25065_How_does_creating_an_index_on_a_WHERE-clause_column_typicall.md>)
- [Intermediate] [What is the impact of joining tables in a different order and does the optimizer reorder joins for you?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Execution_Plans/Q-25095_What_is_the_impact_of_joining_tables_in_a_different_order_an.md>)
- [Intermediate] [When does a query benefit from an index that supports both filtering and ordering to avoid a separate Sort node?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Execution_Plans/Q-24980_When_does_a_query_benefit_from_an_index_that_supports_both_f.md>)
- [Intermediate] [How do you tune a query whose cost blows up because of an unexpected cross product from a missing join condition?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Execution_Plans/Q-25054_How_do_you_tune_a_query_whose_cost_blows_up_because_of_an_un.md>)
- [Deep] [Why can a query with a highly selective filter still do a full scan if the index is very unselective?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Execution_Plans/Q-25120_Why_can_a_query_with_a_highly_selective_filter_still_do_a_fu.md>)
- [Deep] [When an index-only scan still shows high 'Heap Fetches', what is happening and how do you reduce them?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Execution_Plans/Q-24975_When_an_index-only_scan_still_shows_high_Heap_Fetches_what_i.md>)
- [Deep] [How do you approach tuning an analytical query that scans a whole fact table when you can't add indexes freely?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Execution_Plans/Q-25016_How_do_you_approach_tuning_an_analytical_query_that_scans_a_.md>)
- [Deep] [How would you explain to your team the risk of tuning to a specific plan that becomes wrong as data volume grows 10x?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Execution_Plans/Q-25056_How_would_you_explain_to_your_team_the_risk_of_tuning_to_a_s.md>)

## Related concepts

- [Index Tuning in Practice](<../Indexing/Index_Tuning_in_Practice/_index.md>) *(Indexing)*
- [Filtered Indexes and Plan Cache](<../../../TOOLS/SQL_Server/T-SQL/Filtered_Indexes_and_Plan_Cache.md>) *(T-SQL)*
- [EXISTS Semi-joins and Predicate Placement](<../../SQL_Language/Joins_and_Subqueries/EXISTS_Semi-joins_and_Predicate_Placement.md>) *(Joins and Subqueries)*
- [MySQL Optimizer Behavior](<../../../TOOLS/MySQL/MySQL_Platform/MySQL_Optimizer_Behavior.md>) *(MySQL Platform)*
- [Leftmost Prefix Rule](<../../SQL_Language/SQL_Fundamentals/Leftmost_Prefix_Rule.md>) *(SQL Fundamentals)*
- [Index Access Paths](<../../Database_Internals/DBMS_Concepts/Index_Access_Paths.md>) *(DBMS Concepts)*

## References

- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 1 Query Performance Tuning
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 2 Execution Plan Generation And The Query Optimizer
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 5 Statistics, Data Distribution, And Cardinality
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Writing Simple SELECT Queries › Viewing Execution Plans
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Building on Subqueries, Common Table Expressions, and Unions › Thinking About Performance
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Advanced WHERE Clauses › Thinking About Performance
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Introduction: The Importance of Performance Tuning
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Common Operators in Execution Plans
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Practical Example: Comparing Plans
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Importance of Query Rewrites
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Infrastructure Operations › Query optimization
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Using Execution Plans
- *Practical Graph Structures in SQL Server and Azure SQL* — Louis Davidson — A Graph For Testing › Performance Tuning Roundup
- *SQL Server Tutorial* (YouTube series) — Part 62: What to choose for performance   SubQuery or Joins ([watch](https://www.youtube.com/watch?v=dAQhBJMAeqc))

