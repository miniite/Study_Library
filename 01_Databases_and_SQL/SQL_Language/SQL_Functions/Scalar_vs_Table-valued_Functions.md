---
id: "01_Databases_and_SQL/SQL_Language/SQL_Functions/Scalar_vs_Table-valued_Functions"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Functions"
tools: ["SQL_Server"]
---

# Scalar vs Table-valued Functions

**L4 Advanced** · [↑ SQL Functions](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A *scalar* function returns one value per call and is used like an expression; a *table-valued* function (TVF) returns a rowset and is used like a table — and the *inline* TVF (a single parameterized `SELECT`) is the one the optimizer can fully merge into your query.

### 1. The big picture
Built-in functions (`ABS`, `DATEADD`, `UPPER`) cover common needs. When you need your own reusable logic, you write a **user-defined function (UDF)**. Beginning T-SQL lists the motive plainly: "reuse code to simplify development and hide complex logic". The same chapter immediately adds the warning: "use caution… they can negatively affect performance due to the overhead of calling the function for each row."

There are three shapes, and choosing the right one is most of the skill:

| Kind | Returns | Used in | Optimizer treatment (SQL Server) |
|---|---|---|---|
| **Scalar UDF** | one value | `SELECT` list, `WHERE`, computed columns | historically a black box called per row; inlinable from SQL Server 2019 |
| **Inline TVF (iTVF)** | a table, defined by one `SELECT` | `FROM`, `JOIN`, `CROSS/OUTER APPLY` | expanded into the calling query like a parameterized view |
| **Multi-statement TVF (mTVF)** | a table built in a `BEGIN…END` body into a table variable | `FROM`, `APPLY` | opaque; fixed (or 1/100-row) cardinality estimates; usually worse |

In PostgreSQL the same split is by *return type* (`RETURNS int` vs `RETURNS TABLE(...)` / `SETOF`) and *language* (`LANGUAGE sql` functions can be inlined; `plpgsql` cannot). Spark SQL has scalar UDFs too, with the extra cost of serialization between the JVM and Python (Learning Spark).

### 2. Core concepts and vocabulary
- **Scalar UDF** — returns a single atomic value; can be called anywhere an expression is allowed.
- **Table-valued function** — returns a set of rows and columns.
- **Inline TVF** — body is a single `RETURN (SELECT …)`; no `BEGIN…END`.
- **Multi-statement TVF** — declares a return table variable, fills it with several statements, `RETURN`.
- **Deterministic** — same inputs and same database state ⇒ same output (`ABS(-5)`). **Non-deterministic** — may differ (`GETDATE()`, `NEWID()`).
- **`SCHEMABINDING`** — ties a function to the objects it references so they can't be altered underneath it; required for the function to be treated as deterministic/precise in indexed views and indexed computed columns.
- **Side effects** — a UDF may not modify permanent tables or send results to the client; functions are read-only expressions (stored procedures can do DML).
- **Volatility (PostgreSQL)** — `IMMUTABLE` (pure), `STABLE` (constant within a statement), `VOLATILE` (anything).
- **Scalar UDF inlining** — SQL Server 2019+ (compat level 150) rewrite of eligible scalar UDFs into scalar subqueries.
- **`APPLY`** — T-SQL's lateral join: `CROSS APPLY fn(col)` calls a TVF for each outer row.

### 3. How it works, step by step
**Calling a scalar UDF per row (classic behavior):**

1. The optimizer sees `dbo.fn_GetRegion(c.CustomerID)` as an opaque call with a fixed small cost.
2. The executor runs the main plan; for each row it **invokes the function body** as a separate compiled batch (own plan, own context).
3. The function runs its internal statements (often another query), returns a value.
4. The result flows back; a plan containing such a UDF is forced to **run serially** (no parallelism) in the old model.

**Calling an inline TVF:**

1. At compile time the engine **substitutes the function's `SELECT` into the calling query** (like a view with parameters).
2. The whole thing is optimized as one tree — join reordering, predicate push-down, index seeks, parallelism all apply.
3. With `CROSS APPLY` it behaves like a correlated subquery that the optimizer can often turn into a join.

**Calling a multi-statement TVF:**

1. The engine executes the body *first*, filling a table variable.
2. The result is treated as a table with a *guess* (1 row up to SQL Server 2012, 100 rows after, and since 2017 an interleaved-execution estimate for mTVFs) and **no column statistics**.
3. The outer plan is built on that guess — good only when the guess is close.

### 4. Worked example
```sql
-- 1. Scalar UDF
CREATE FUNCTION dbo.fn_NetPrice (@price money, @disc decimal(4,2))
RETURNS money WITH SCHEMABINDING
AS BEGIN
    RETURN @price * (1 - COALESCE(@disc, 0));
END;
GO
SELECT OrderID, dbo.fn_NetPrice(Price, Discount) AS Net FROM dbo.Orders;

-- 2. Inline TVF: orders for a customer in a date range
CREATE FUNCTION dbo.fn_CustomerOrders (@cust int, @from date, @to date)
RETURNS TABLE
AS RETURN
(
    SELECT OrderID, OrderDate, Price
    FROM   dbo.Orders
    WHERE  CustomerID = @cust AND OrderDate >= @from AND OrderDate < @to
);
GO
SELECT c.Name, o.OrderID, o.Price
FROM   dbo.Customers c
CROSS APPLY dbo.fn_CustomerOrders(c.CustomerID, '2025-01-01', '2026-01-01') o;

-- 3. Multi-statement TVF (same result, usually slower)
CREATE FUNCTION dbo.fn_CustomerOrders_m (@cust int)
RETURNS @t TABLE (OrderID int, OrderDate date, Price money)
AS BEGIN
    INSERT @t SELECT OrderID, OrderDate, Price FROM dbo.Orders WHERE CustomerID = @cust;
    RETURN;
END;
```

How to read it: in the actual execution plan, example 2 shows **no function operator** — you see the `Orders` seek/join directly, because the iTVF was expanded. Example 3 shows a *Table Scan of `@t`* with an estimated 1 (or 100) rows regardless of reality; look at "Estimated vs Actual Number of Rows" to see the mismatch. For example 1, check `SET STATISTICS XML`/the plan for "ContainsInlineScalarTsqlUdfs" (inlined) or, when not inlined, higher CPU than elapsed time and a serial plan.

### 5. Under the hood
- **Why scalar UDFs hurt (pre-2019).** Fritchey's *Query Performance Tuning* lists three causes: scalar functions aren't relational so the optimizer **cannot cost them properly**; each UDF statement is **compiled independently** so it can't be optimized with the surrounding query; and UDFs run **serially, with no parallelism**. They also hide I/O: `SET STATISTICS IO` doesn't show reads performed inside the function, so the query looks cheap while it is not.
- **Scalar UDF inlining** (SQL Server 2019, compat ≥150). Eligible functions are turned into a scalar subquery and merged, so "most of the standard issues with scalar functions are eliminated". It isn't universal: functions that use `GETDATE()`-like time-dependent intrinsics, table variables, `WITH EXECUTE AS`, recursion, or certain constructs are excluded. Check `sys.sql_modules.is_inlineable`.
- **mTVF cardinality fixes.** SQL Server 2017 Interleaved Execution pauses optimization to execute the mTVF and uses the real row count for the rest of the plan; this fixes many cases but not all.
- **Determinism and indexing.** An index on a computed column (or an indexed view) requires a *deterministic* expression; `GETDATE()` or a non-schema-bound UDF is rejected (Expert Performance Indexing). A `PERSISTED` computed column stores the computed result and can be indexed, trading storage for avoiding recomputation.
- **PostgreSQL.** `LANGUAGE sql` functions with simple bodies are inlined into the calling query; `plpgsql` is never inlined. Volatility labels drive planning: `IMMUTABLE` functions on constants are folded at plan time and are allowed in expression indexes. Mislabeling a function that reads a setting or a table as `IMMUTABLE` makes the planner cache or fold a result that should vary — the cure is `STABLE` (or `VOLATILE`).
- **Functions can't change data.** That is by design: a function is evaluated in the middle of a query; permitting DML would make query results order- and plan-dependent. (PostgreSQL `VOLATILE` functions *can* do DML; T-SQL UDFs can't touch permanent tables.)

### 6. Performance and design implications
- **Default to an inline TVF** for reusable set logic; it's "just a parameterized view".
- **Replace scalar UDFs with set logic** where possible: a `JOIN` to a lookup table instead of `fn_GetRegion(CustomerID)`, or the formula written inline. Ultimate T-SQL gives exactly this rewrite.
- **Keep UDFs out of `WHERE`/`JOIN` predicates on columns**: the function hides the column, so no seek and no statistics (the same SARGability issue as `RTRIM(col) = …`, which jumped from 2 reads to 99+ in the Expert Indexing example).
- **For indexability**, make the computation a deterministic persisted computed column (or a function-based index in PostgreSQL/Oracle) and index that.
- **Avoid mTVFs** unless the result is small and fixed; otherwise write an inline TVF or a temp table + query.
- **Measure**: compare `CPU time` vs `elapsed time`, use Extended Events / Query Store, `sys.dm_exec_function_stats` (SQL Server 2016+), and test with and without the call.
- **Lookup table vs function for code→label mapping**: a table with a PK joins cheaply, has statistics, and can be updated without a deployment; a function only wins when the logic is genuinely computational.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE FUNCTION … RETURNS type` | Scalar UDF | Small deterministic calculations |
| `CREATE FUNCTION … RETURNS TABLE AS RETURN (SELECT…)` | Inline TVF | Reusable parameterized query logic |
| `RETURNS @t TABLE (…) AS BEGIN … END` | Multi-statement TVF | Rare; procedural construction |
| `WITH SCHEMABINDING` | Locks referenced objects; helps determinism | Functions used in computed columns / indexed views |
| `CROSS APPLY` / `OUTER APPLY` | Call TVF per outer row | Top-N per group, parameterized subqueries |
| `OBJECTPROPERTY(id,'IsDeterministic')`, `sys.sql_modules.is_inlineable` | Introspection | Verify function properties |
| `IMMUTABLE / STABLE / VOLATILE` (PostgreSQL) | Volatility labels | Enable folding, expression indexes |
| `ALTER DATABASE … SET COMPATIBILITY_LEVEL = 150` | Enables scalar UDF inlining | Upgrading from older compat levels |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Inline TVF for reusable filtered rowsets | You need multiple procedural steps (consider a procedure/temp table) |
| Scalar UDF for a tiny pure calculation, on SQL Server 2019+ and inlinable | Large scans on older compat levels, or non-inlinable bodies |
| Built-in function or inline expression | Duplicating a complex formula in many queries (maintenance) |
| Lookup-table join for mapping | Logic that is truly computed |
| Persisted computed column + index | Non-deterministic expressions |

### 9. Common mistakes
- Calling a scalar UDF that runs a query inside a SELECT over millions of rows → hidden N+1 → inline TVF/join/inline expression.
- Using a mTVF "because it's a table function" → 1/100-row guess, bad join choices → inline TVF.
- Believing a UDF can update a table → error "side-effecting operator within a function" → use a procedure.
- Using `GETDATE()` in a computed column you want indexed → error → pass it as a parameter or compute at query time.
- Marking a PostgreSQL function `IMMUTABLE` when it reads a table or `current_setting` → stale results → `STABLE`/`VOLATILE`.
- Not schema-binding functions used in indexed objects → can't be indexed or becomes imprecise.

### 10. 🎯 Interview kit
**30-second answer:** "A scalar function returns one value and is used as an expression; a table-valued function returns rows and is used in `FROM`. There are inline TVFs (single SELECT, expanded like a view and fully optimizable) and multi-statement TVFs (table variable, poor estimates). Scalar UDFs have historically forced row-by-row, serial execution; SQL Server 2019 can inline many of them."

**Structure a longer answer:** definitions → the optimizer's treatment of each → measured performance consequences → how I'd refactor (inline TVF/join/computed column) → determinism/SCHEMABINDING for indexing.

**Likely follow-ups:**
- *Scalar vs table-valued?* → return type and where usable.
- *Why is a scalar UDF slow?* → per-row invocation, no costing, no parallelism, hidden I/O.
- *Inline vs multi-statement TVF?* → expansion vs black box with bad estimates.
- *Can a function change data?* → no (T-SQL); use a procedure.
- *What makes a function indexable in a computed column?* → deterministic, precise, schema-bound.
- *PostgreSQL volatility?* → IMMUTABLE/STABLE/VOLATILE and the cost of mislabelling.
- *Function vs procedure from app code?* → function returns a value/rowset and composes in queries; procedure can do DML/transactions and returns status/result sets.

**Signals interviewers look for:** you reach for inline TVFs and set-based rewrites, know the 2019 inlining feature and its limits, and mention determinism for indexing.

### 11. Check yourself
- [ ] Rewrite a scalar UDF used in `WHERE` as an inline TVF or join.
- [ ] Explain from a plan why a mTVF estimate of 1 or 100 rows can ruin a join.
- [ ] List three reasons scalar UDF inlining might not apply.
- [ ] State the conditions for indexing a computed column that calls a function.
- [ ] Choose `IMMUTABLE`, `STABLE` or `VOLATILE` for three example functions and justify.
<!-- explainer:end -->

## Implementations (tools)

- [SQL Server](<../../../TOOLS/SQL_Server/_index.md>)

## Practice questions

- [Basic] [What is a user-defined function (UDF) and when would you create one?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41285_What_is_a_user-defined_function_UDF_and_when_would_you_creat.md>)
- [Basic] [What does the EXP and LOG functions compute and where might they be used?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41335_What_does_the_EXP_and_LOG_functions_compute_and_where_might_.md>)
- [Intermediate] [What is the difference between calling a function and calling a stored procedure from application code?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41346_What_is_the_difference_between_calling_a_function_and_callin.md>)
- [Deep] [What is the difference between a deterministic and a non-deterministic function? Give an example of each.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41289_What_is_the_difference_between_a_deterministic_and_a_non-det.md>)
- [Deep] [You added a PERSISTED computed column derived from a function and want to index it. What conditions must the function meet for the column to be indexable?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41160_You_added_a_PERSISTED_computed_column_derived_from_a_functio.md>)
- [Deep] [You marked a PL/pgSQL function IMMUTABLE but its result depends on a config setting and returns stale cached values. What went wrong and how do you fix the volatility labeling?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41167_You_marked_a_PL_pgSQL_function_IMMUTABLE_but_its_result_depe.md>)
- [Deep] [How do CLR/user functions interact with the schema-binding option, and why does SCHEMABINDING matter for indexing and determinism guarantees?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41223_How_do_CLR_user_functions_interact_with_the_schema-binding_o.md>)
- [Deep] [You need to hash PII with a function like HASHBYTES/MD5/SHA2 for pseudonymization. Discuss determinism, collision risk, salting, and index implications.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41253_You_need_to_hash_PII_with_a_function_like_HASHBYTES_MD5_SHA2.md>)

## Related concepts

- [Dynamic SQL](<../../Database_Internals/DBMS_Concepts/Dynamic_SQL.md>) *(DBMS Concepts)*
- [CTEs in SQL Server](<../../../TOOLS/SQL_Server/SQL_Server_Platform/CTEs_in_SQL_Server.md>) *(SQL Server Platform)*
- [Stored Procedure Performance and Design](<../../Database_Objects/Stored_Procedures/Stored_Procedure_Performance_and_Design.md>) *(Stored Procedures)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Functions in SQL Server
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Historical Context
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Scalar Functions
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Deterministic vs. Non-Deterministic Functions
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — User-Defined Functions
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — User-Defined Functions › Creating User-Defined Scalar Functions
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — User-Defined Functions › Using Table-Valued User-Defined Functions
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — User-Defined Data Types
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Spark SQL and DataFrames Interacting with External Data Sources › Option 2: User-Defined Function
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Spark SQL and DataFrames Interacting with External Data Sources › Built-in Functions for Complex Data Types
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Spark SQL and DataFrames Interacting with External Data Sources › Higher-Order Functions
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Arbitrary Stateful Computations
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page Compression › Computed Columns
- *Fuzzy Data Matching with SQL* — Jim Lehmer — Function Junction › Chapter 2. Function Junction
- *Data Engineering Design Patterns* — Bartosz Konieczny — Error Management Design Patterns › Error-safe functions
- *SQL Server Tutorial* (YouTube series) — Part 29: Mathematical functions in sql server ([watch](https://www.youtube.com/watch?v=gJ4PDXFHjbc))
- *SQL Server Tutorial* (YouTube series) — Part 30: Scalar user defined functions in sql server ([watch](https://www.youtube.com/watch?v=OV5CquR1Svo))
- *SQL Server Tutorial* (YouTube series) — Part 33: Important concepts related to functions in sql server ([watch](https://www.youtube.com/watch?v=WNoTgfg3mGc))

