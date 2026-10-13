---
id: "01_Databases_and_SQL/SQL_Language/SQL_Functions/UDF_Performance"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Functions"
tools: ["SQL_Server"]
---

# UDF Performance

**L4 Advanced** · [↑ SQL Functions](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Functions are cheap in isolation and expensive in aggregate — a function applied to a column in `WHERE`/`JOIN`/`GROUP BY` blocks index seeks, a scalar UDF can force row-by-row serial execution, and window/aggregate functions have their own memory and sort costs; diagnose with the plan, then rewrite so the engine works on set-based, sargable expressions.

### 1. The big picture
"Function" in performance conversations means four different things, and each fails differently:

1. **Built-in scalar functions on columns** (`YEAR(d)`, `UPPER(s)`, `COALESCE(x,0)`) — make a predicate **non-sargable** (cannot seek an index).
2. **User-defined scalar/multi-statement functions** — per-row execution, poor costing, no parallelism (older SQL Server), hidden I/O.
3. **Aggregate and window functions** (`COUNT(DISTINCT)`, `ROW_NUMBER() OVER …`) — sort/hash work and memory grants that can spill.
4. **Expressions in grouping/joining** (`DATE_TRUNC`, `DATEDIFF` in a join predicate) — prevent index-ordered grouping and merge joins.

The three T-SQL books agree on the headline. Ultimate T-SQL: scalar functions in `SELECT`/`WHERE` "severely degrade performance and limit the query optimizer's ability to parallelize". Expert Performance Indexing shows `WHERE RTRIM(FirstName) = 'Gustavo'` turning an index seek (2 reads) into a scan (99+ reads). Fritchey's Query Performance Tuning explains why: the optimizer doesn't cost scalar functions, compiles each function statement separately, and runs them serially.

### 2. Core concepts and vocabulary
- **SARGable** — "Search ARGument able": a predicate of the form `column <op> constant/parameter` that an index can seek on. Applying a function to the column breaks it.
- **Index seek vs scan** — seek navigates the B-tree to matching rows; scan reads the whole index/table.
- **Row-by-row ("RBAR")** — executing logic once per row rather than once per set.
- **Scalar UDF inlining** — SQL Server 2019+ transformation of eligible scalar UDFs into scalar subqueries.
- **Constant folding** — evaluating a deterministic expression on constants once at compile time (`WHERE d >= DATEADD(day,-7,'2025-01-01')`).
- **Expression/function-based index** — an index on a computed expression (PostgreSQL/Oracle/MySQL 8) or on a computed column (SQL Server) that matches the function in the predicate.
- **Memory grant** — workspace memory a query reserves for sort/hash; underestimation causes **spills** to tempdb.
- **Window frame** — the set of rows a window function aggregates over relative to the current row.

### 3. How it works, step by step
**Why a function on a column breaks an index seek:**

1. An index stores *column values* in sorted order.
2. The predicate `WHERE UPPER(name) = 'SMITH'` asks about *transformed* values, which aren't stored or sorted.
3. The engine can't binary-search; it must read every row, apply the function, compare.
4. Also, statistics describe `name`, not `UPPER(name)`, so the row estimate is a guess — leading to bad join/memory choices downstream.

**Fix options (in order of preference):**

1. **Rewrite the predicate** so the column is bare: `WHERE order_date >= '2025-01-01' AND order_date < '2026-01-01'` instead of `YEAR(order_date) = 2025`.
2. **Normalize the data** at write time (store `lower(email)`, store the date key).
3. **Index the expression**: PostgreSQL `CREATE INDEX ON users (lower(email));`, SQL Server `ALTER TABLE users ADD email_lc AS LOWER(email) PERSISTED; CREATE INDEX ix ON users(email_lc);`, or a case-insensitive collation.
4. As a last resort accept the scan, but reduce the work with a covering narrow index.

### 4. Worked example
```sql
-- 1. Non-sargable
SELECT COUNT(*) FROM Sales.Orders WHERE YEAR(OrderDate) = 2025 AND MONTH(OrderDate) = 3;
-- plan: Index Scan, estimated rows ≈ 30% guess, reads = whole index

-- 2. Sargable
SELECT COUNT(*) FROM Sales.Orders
WHERE OrderDate >= '2025-03-01' AND OrderDate < '2025-04-01';
-- plan: Index Seek (range), accurate estimate from histogram
```

Check with:
```sql
SET STATISTICS IO, TIME ON;       -- logical reads, CPU vs elapsed
-- or look at the actual plan: seek vs scan, "Estimated vs Actual Rows"
```

**A scalar UDF at scale (SQL Server, compat < 150):**
```sql
SELECT o.OrderID, dbo.ufnGetProductStandardCost(o.ProductID, o.OrderDate) AS Cost
FROM   Sales.BigOrders o;   -- 50 million rows, 20 minutes
```
Diagnosis: elapsed ≈ CPU (serial), plan shows a *Compute Scalar* with the UDF, no parallelism zones, and `STATISTICS IO` reports tiny reads because the function's reads are hidden. Confirm with `sys.dm_exec_function_stats` (execution_count ≈ row count, total_worker_time dominant) or an Extended Events trace on `sp_statement_completed`. Fixes: (a) upgrade compatibility level to 150 so the function is inlined; (b) rewrite as an inline TVF with `CROSS APPLY`; (c) pre-join the lookup (`ProductCostHistory`) with a range join; (d) precompute into a table.

```sql
-- inline TVF rewrite
CREATE FUNCTION dbo.fn_StdCost(@ProductID int, @OrderDate datetime)
RETURNS TABLE AS RETURN
 (SELECT TOP (1) StandardCost
  FROM Production.ProductCostHistory
  WHERE ProductID=@ProductID AND @OrderDate BETWEEN StartDate AND COALESCE(EndDate,'9999-12-31'));
GO
SELECT o.OrderID, c.StandardCost
FROM Sales.BigOrders o
OUTER APPLY dbo.fn_StdCost(o.ProductID, o.OrderDate) c;   -- optimizer can merge and parallelize
```

### 5. Under the hood
- **Per-call overhead.** A non-inlined scalar UDF call sets up its own execution context, runs its statements, and returns: microseconds of overhead per row, plus whatever the body does. At 50 million rows the overhead alone is minutes.
- **Why no parallelism.** Using a scalar UDF (not inlined) marks the plan as non-parallelizable in older versions; mTVFs and UDFs that access data have this property. SQL Server 2019 inlining removes it for eligible functions; Kathi Kellenberger's chapter shows inline TVFs become joins and get better plans.
- **Statistics invisibility.** When you write `WHERE f(col) = x`, the optimizer uses density guesses (e.g. 10% for equality on a computed expression, 30% for inequality) instead of the histogram.
- **Implicit conversions are a hidden function.** Comparing a `varchar` column with an `nvarchar` parameter wraps the column in `CONVERT_IMPLICIT`, which turns a seek into a scan (data-type precedence). Check for the "PlanAffectingConvert" warning; fix by matching types.
- **`COALESCE`/`ISNULL` in predicates** (`WHERE COALESCE(col,0)=0`) are sargability killers; rewrite as `col = 0 OR col IS NULL`. `COALESCE` over several correlated subqueries expands to a `CASE`, and each `WHEN` can evaluate its subquery separately (and, with the `CASE` expansion, even twice), so cost multiplies per row.
- **Aggregates.** `COUNT(DISTINCT col)` needs a sort or hash of the distinct values (memory proportional to cardinality); `APPROX_COUNT_DISTINCT` uses HyperLogLog-style sketches for ~1–2% error with tiny memory. `PARTITION BY` in window functions requires sorting by partition+order keys; a missing supporting index causes a Sort operator whose memory grant is estimated from cardinality — wrong estimates → spill to tempdb ("Sort Warning").
- **Window frames.** The default frame with `ORDER BY` is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`, which is why `LAST_VALUE` returns the current row; and `RANGE` (default) uses an on-disk spool in SQL Server, while `ROWS` uses an in-memory one — specify `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` for running totals; it's often 5–10× faster.

### 6. Performance and design implications
- **Principle:** keep the column bare on one side of the comparison and put the function on the other (constant) side; the engine can fold constants.
- **Dates:** half-open ranges instead of `YEAR()`, `MONTH()`, `CONVERT(date, col)`; precomputed `date_key`/`dim_date` joins; persisted truncated column if you group by `DATE_TRUNC` all the time.
- **Strings:** use case-insensitive collation or a normalized column rather than `UPPER()`/`LOWER()` in predicates; avoid leading wildcards `LIKE '%x'`.
- **Aggregates:** `FILTER (WHERE …)` or `SUM(CASE …)` for conditional aggregates — equal plans, FILTER is more readable; use `GROUPING SETS/ROLLUP` to compute many group levels in one pass instead of `UNION ALL` of several scans.
- **Windows:** index on `(partition keys, order keys) INCLUDE (needed columns)` to avoid the sort; narrow the partition size; reuse a named `WINDOW` clause for readability (the plan is the same as repeated inline `OVER` definitions).
- **Top-N per group / dedupe:** `ROW_NUMBER() OVER (PARTITION BY k ORDER BY ts DESC) = 1` is usually better than `GROUP BY` + self-join because it reads the table once. Snowflake/BigQuery `QUALIFY` filters on a window result directly; elsewhere wrap in a CTE/subquery.
- **JSON functions** (`JSON_VALUE`, `JSON_EXTRACT`) are non-sargable on the raw column; create a computed/generated column from the path and index that.
- **Hashing PII** (`HASHBYTES`, `SHA2`) is deterministic — equal inputs give equal outputs (good for joins/indexes, bad for privacy without a secret salt). Index the stored hash, not the function call.
- **Floating point sums** over `FLOAT` can differ run-to-run because parallel partial sums add in different orders; use `DECIMAL` for money.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SET STATISTICS IO, TIME ON` | Logical reads, CPU/elapsed | First measurement |
| Actual execution plan / `EXPLAIN (ANALYZE, BUFFERS)` | Seek vs scan, estimates vs actuals, spills | Diagnosing any of the above |
| `sys.dm_exec_function_stats` | Per-function call counts and CPU (SQL Server 2016+) | Prove a UDF is the bottleneck |
| `sys.sql_modules.is_inlineable` | Can SQL Server inline this UDF? | Before/after compatibility upgrade |
| `COMPATIBILITY_LEVEL = 150+` | Enables scalar UDF inlining & IQP features | Modernization |
| `CREATE INDEX … ON t (lower(col))` | Expression index | PostgreSQL/Oracle/MySQL 8 |
| `ALTER TABLE … ADD c AS expr PERSISTED` + index | Indexed computed column | SQL Server |
| `OPTION (RECOMPILE)` / `OPTIMIZE FOR` | Plan control | Parameter-dependent function predicates |
| `APPROX_COUNT_DISTINCT` | Sketch-based distinct count | Dashboards where ±2% is fine |
| `WITH (SCHEMABINDING)` | Required for deterministic/indexable UDF | Indexed views and computed columns |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Range predicate on the raw column | Business rule truly needs the transformed value (use persisted computed column) |
| Expression index | Write-heavy tables where each extra index costs inserts |
| Persisted computed column | Large wide columns (storage) |
| Inline TVF / join rewrite | Function logic changes often and many callers depend on it (but inline TVFs still centralize it) |
| Precomputed dimension (`dim_date`) | Ad-hoc one-off queries |
| `ROWS` frame for running totals | Value-based frame semantics are needed (`RANGE` with ties) |
| Approximate distinct | Financial or compliance counts |

### 9. Common mistakes
- Wrapping the column: `WHERE YEAR(order_date)=2025`, `WHERE UPPER(name)=…`, `WHERE COALESCE(col,0)=0` → scans, bad estimates → bare column.
- Trusting `STATISTICS IO` for a UDF query → it omits function reads → compare CPU/elapsed and DMVs.
- Calling a scalar UDF in a join predicate (`ON dbo.fn(a)=b`) → nested loop + scan, no parallelism → precompute or APPLY.
- Using `DATEDIFF` in a join condition → the optimizer can't use an index → compare with a range using `DATEADD` on the constant side.
- Adding `ORDER BY` inside a window frame without index → sort + spill → supporting index; check memory grant.
- Not specifying a frame for `LAST_VALUE` → returns current row → `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`.
- `COUNT(DISTINCT)` on high-cardinality columns in big group-bys → memory pressure → pre-aggregate or approximate.

### 10. 🎯 Interview kit
**30-second answer:** "A function on an indexed column in a predicate makes it non-sargable, so the index can't be sought and statistics are lost; scalar UDFs add per-row calls and serial plans. I diagnose with the actual plan and STATISTICS IO/TIME, then rewrite so the column is bare, use an inline TVF or join instead of the scalar UDF, or index the expression."

**Structure a longer answer:** symptom → measurement (plan, reads, CPU vs elapsed) → mechanism (non-sargable, per-row UDF, spill) → fix hierarchy (rewrite, persisted/expression index, set-based) → trade-off (index write cost) → verification.

**Likely follow-ups:**
- *Why does `UPPER(col)=…` ignore the index?* → index stores raw values; use expression index or collation.
- *`WHERE YEAR(d)=2025` vs range for a partitioned table?* → range allows partition elimination and a seek; function forces scanning all partitions.
- *A 20-minute scalar UDF query — what do you do?* → prove with DMVs, inline/TVF/join/precompute, compat level 150.
- *Running total performance?* → window with `ROWS` frame vs self-join (O(n²)).
- *`COUNT(DISTINCT)` cost?* → hash/sort of distinct values; approximate alternative.
- *Named WINDOW clause?* → readability; same plan.

**Signals interviewers look for:** you start from measurement, name the mechanism (sargability, costing, parallelism), and offer layered fixes with the cost of each.

### 11. Check yourself
- [ ] Take `WHERE CONVERT(date, ts) = '2025-03-01'` and rewrite it sargably.
- [ ] Describe three reasons a scalar UDF is slow on older SQL Server and how 2019 changes that.
- [ ] Show how to index `LOWER(email)` in PostgreSQL and in SQL Server.
- [ ] Explain why `LAST_VALUE()` returned the current row and the frame that fixes it.
- [ ] Read a plan and identify a sort spill from a `PARTITION BY` window.
<!-- explainer:end -->

## Implementations (tools)

- [SQL Server](<../../../TOOLS/SQL_Server/_index.md>)

## Practice questions

- [Intermediate] [How can you count how many orders each customer placed and only show those with more than 5 orders?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41328_How_can_you_count_how_many_orders_each_customer_placed_and_o.md>)
- [Intermediate] [Why can DATE_TRUNC or TRUNC(date) in a GROUP BY be slower than pre-computing a date key, and how do you decide when to persist the truncated value?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41231_Why_can_DATE_TRUNC_or_TRUNC_date_in_a_GROUP_BY_be_slower_tha.md>)
- [Deep] [You have a scalar UDF called in the SELECT list of a 50-million-row query and it runs for 20 minutes. How do you diagnose whether the function is the bottleneck and what would you do about it?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41155_You_have_a_scalar_UDF_called_in_the_SELECT_list_of_a_50-mill.md>)
- [Deep] [You have LAST_VALUE() returning the current row instead of the partition's last value. Explain the frame issue and the correct frame specification to fix it.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41173_You_have_LAST_VALUE_returning_the_current_row_instead_of_the.md>)
- [Deep] [A datetime function like DATEDIFF is used in a join predicate and the optimizer picks a nested loop with a scan. How would you refactor to make the join index-friendly?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41192_A_datetime_function_like_DATEDIFF_is_used_in_a_join_predicat.md>)
- [Deep] [You must call a function that hits an external service inside a query and it hangs the batch. Why are side-effecting functions in the data path an anti-pattern, and what's the fix?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41224_You_must_call_a_function_that_hits_an_external_service_insid.md>)
- [Deep] [A HAVING clause filters on an aggregate function while WHERE could filter earlier. Explain the logical processing order and why moving a predicate from HAVING to WHERE changed the result AND performance.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41237_A_HAVING_clause_filters_on_an_aggregate_function_while_WHERE.md>)
- [Deep] [A batch uses SYSDATE inside an INSERT for millions of rows and audit timestamps vary slightly per row. Is that a bug? How do you make the timestamp consistent per statement?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41261_A_batch_uses_SYSDATE_inside_an_INSERT_for_millions_of_rows_a.md>)

## Related concepts

- [Scalar Subqueries](<../SQL_Fundamentals/Scalar_Subqueries.md>) *(SQL Fundamentals)*
- [MySQL Optimizer Behavior](<../../../TOOLS/MySQL/MySQL_Platform/MySQL_Optimizer_Behavior.md>) *(MySQL Platform)*
- [Stored Procedure Performance and Design](<../../Database_Objects/Stored_Procedures/Stored_Procedure_Performance_and_Design.md>) *(Stored Procedures)*

## References

- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 8 Query Recompilation
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 21 Intelligent Query Processing
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page Compression › Scalar Functions
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 2: Data Types, NULLs and Conversions › Performance Considerations
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Structure
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Performance Considerations
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 7: Dynamic SQL, Triggers, and Table-Valued Functions (TVFs) › Practical Use Cases
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Thinking About Performance › Solutions to Exercise 4-7: Using Functions in the WHERE and ORDER BY Clauses
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Discovering Windowing Functions › Ranking Functions and the OVER Clause
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Thinking About Performance
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — Refactoring Data Access You'll recall we presented a design for the Reservation Service in Chapter
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — T-SQL for data analysis
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Programmable objects

