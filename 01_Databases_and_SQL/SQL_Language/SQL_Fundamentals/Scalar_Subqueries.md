---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Scalar_Subqueries"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: ["SQL_Server", "MySQL"]
---

# Scalar Subqueries

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A scalar subquery is a `SELECT` in parentheses that must return at most one row and one column, so it can be used anywhere a single value is allowed; the engine treats zero rows as `NULL` and more than one row as an error.

### 1. The big picture
A subquery is a query nested inside another. Three shapes matter:

| Shape | Returns | Where it is used |
|---|---|---|
| **Scalar** | one value | `SELECT` list, `WHERE x = (…)`, `SET col = (…)`, expressions |
| **Multi-valued (column)** | a set of values | `IN (…)`, `ANY/ALL`, `EXISTS` |
| **Table (derived table / CTE)** | a whole result set | `FROM (…) AS d`, `WITH cte AS (…)` |

Subqueries can also be **self-contained** (independent of the outer query; evaluated once logically) or **correlated** (refer to a column of the outer query; evaluated per outer row, logically). The optimizer is free to rewrite them, so the *logical* meaning is what you reason about and the *plan* is what you verify.

Scalar subqueries are the quickest way to say "compare to the overall average" or "fetch the latest related value", but they are also a classic source of silent bugs (zero rows → `NULL`) and performance problems (accidental row-by-row execution).

### 2. Core concepts and vocabulary
- **Scalar subquery** — returns one column and at most one row.
- **Self-contained subquery** — can run on its own.
- **Correlated subquery** — references outer columns, e.g. `WHERE o.customer_id = c.customer_id`.
- **Derived table** — a subquery in `FROM` with a mandatory alias.
- **CTE (`WITH`)** — a named, query-scoped derived table; improves readability, not necessarily performance.
- **`EXISTS`** — predicate true if the subquery returns any row; ignores the select list.
- **Three-valued logic** — predicates evaluate to TRUE, FALSE or UNKNOWN (from `NULL`); `WHERE` keeps only TRUE.
- **Unnesting / decorrelation** — the optimizer converting a correlated subquery to a join.

### 3. How it works, step by step
**Rule 1: cardinality.** The subquery is evaluated; if it returns zero rows its value is `NULL`; if it returns exactly one row that is the value; if it returns more than one row you get an error (SQL Server: *Subquery returned more than 1 value*; PostgreSQL: *more than one row returned by a subquery used as an expression*). The error may appear only after the data changes, which is why a query that "worked for months" suddenly fails.

**Rule 2: scope.** An inner query can see outer-query columns, never the reverse. Table aliases resolve to the nearest enclosing scope, so an unqualified column name inside a subquery can silently bind to the *inner* table instead of the outer one you meant.

**Rule 3: `NULL` handling.** `x = (scalar)` is UNKNOWN when the scalar is `NULL`, and the row is dropped.

Logical evaluation of `SELECT … FROM A WHERE A.col > (SELECT AVG(col) FROM B)`:
1. Evaluate the subquery once → one number (or `NULL`).
2. For each row of `A`, compare, keep rows where TRUE.

Logical evaluation of a correlated scalar subquery in the select list:
1. For each outer row, bind the correlated column(s).
2. Run the inner query, require ≤ 1 row, take the value (or `NULL`).
3. Emit the outer row with that value.

### 4. Worked example
```sql
-- Orders above the overall average order value (self-contained scalar subquery)
SELECT order_id, customer_id, total
FROM   orders
WHERE  total > (SELECT AVG(total) FROM orders);

-- Latest order date per customer (correlated scalar subquery in SELECT list)
SELECT c.customer_id, c.name,
       (SELECT MAX(o.order_date)
        FROM   orders o
        WHERE  o.customer_id = c.customer_id) AS last_order_date
FROM   customers c;
```

The second query returns `NULL` for customers with no orders, because `MAX` over an empty set is `NULL`; it never errors because an aggregate always returns exactly one row. Replace `MAX(o.order_date)` with `o.order_date` and any customer with two orders produces the "more than one row" error.

Safer or faster alternatives:

```sql
-- Same result with a join + aggregate (set-based; planner can use hash/merge join)
SELECT c.customer_id, c.name, o.last_order_date
FROM   customers c
LEFT   JOIN (SELECT customer_id, MAX(order_date) AS last_order_date
             FROM orders GROUP BY customer_id) o
       ON o.customer_id = c.customer_id;

-- Several values from the same related row: LATERAL / APPLY instead of repeating subqueries
SELECT c.customer_id, x.order_id, x.order_date, x.total
FROM   customers c
OUTER  APPLY (SELECT TOP (1) order_id, order_date, total        -- SQL Server
              FROM orders o WHERE o.customer_id = c.customer_id
              ORDER BY order_date DESC) x;
-- PostgreSQL: LEFT JOIN LATERAL (SELECT ... ORDER BY order_date DESC LIMIT 1) x ON true
```

The `NOT IN` trap:

```sql
-- Customers with no orders — WRONG if orders.customer_id can be NULL
SELECT * FROM customers WHERE customer_id NOT IN (SELECT customer_id FROM orders);
-- One NULL in the list makes every comparison UNKNOWN → zero rows returned.

-- Correct and NULL-safe
SELECT * FROM customers c
WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id);
```

`SELECT 1` inside `EXISTS` is a convention: `EXISTS` only tests for the presence of rows, so the select list is never evaluated (`SELECT *` is equally fine).

### 5. Under the hood
- **Decorrelation.** Modern optimizers rewrite `WHERE EXISTS (correlated)` into a *semi join* and `NOT EXISTS` into an *anti join*; `IN (subquery)` also becomes a semi join. The join algorithm (hash, merge, nested loop with index seek) is then picked on cost. A correlated scalar subquery in the select list is often rewritten into an outer join with an aggregate, but not always: complex ones remain "per outer row" and appear in plans as a nested-loops/`Subplan` with a repeated index seek.
- **Plan reading.** In SQL Server look for *Nested Loops (Left Outer Join)* over a *Stream Aggregate* with a seek on the inner side and the "Number of Executions" equal to outer row count. In PostgreSQL look for `SubPlan` and `loops=` in `EXPLAIN ANALYZE`.
- **Assert operator.** SQL Server adds an *Assert* operator to a scalar subquery that is not provably single-row, to raise error 512 at runtime. Using `MAX`, `TOP (1)` or a unique-key equality lets the optimizer prove it's single-row and drop the assert.
- **CTEs.** Treated as inlined views in SQL Server (re-evaluated at each reference). PostgreSQL 12+ inlines non-recursive, single-reference CTEs; `MATERIALIZED` forces one evaluation. A CTE is not a temporary table.
- **Huge `IN` lists.** A literal `IN (…8000 values…)` bloats parse time, produces a plan per distinct list length (cache pollution), and can overflow optimizer limits. Put the values in a temp table / table-valued parameter / `VALUES` list / array and join.

### 6. Performance and design implications
- Index the correlated column (`orders(customer_id, order_date)`) so each probe is a seek, and the cost is roughly *outer rows × log(inner)*.
- Prefer one set-based join to several scalar subqueries from the same table; ten subqueries on `orders` can mean ten probes per customer.
- Aggregate-valued scalar subqueries are safe on cardinality, non-aggregate ones need a unique guarantee or `TOP 1 … ORDER BY`.
- `SELECT *` inside views/subqueries pins you to column positions and widens I/O; list columns explicitly.
- Name derived tables and CTE steps meaningfully; they are the documentation of the query.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `EXISTS` / `NOT EXISTS` | Presence test, NULL-safe | Filtering by related rows |
| `IN (subquery)` | Membership | Small, NULL-free lists |
| `= (SELECT MAX(…))` | Scalar comparison | Compare to a single computed value |
| `CROSS/OUTER APPLY`, `LEFT JOIN LATERAL` | Per-row subquery returning multiple columns | Top-N per group |
| `WITH cte AS (…)` | Name an intermediate result | Readability, multi-step logic |
| `ANY / ALL` | Compare to a set | `> ALL (…)` meaning greater than every value |
| `SET STATISTICS IO ON` / `EXPLAIN (ANALYZE)` | Count reads/loops | Detect per-row subplans |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Scalar subquery for a single aggregate compared in `WHERE` | Same subquery repeated in many places or in a hot select list |
| `EXISTS` for "has related rows" | `IN` with a nullable subquery and negation (`NOT IN`) |
| Join + pre-aggregated derived table for many columns | Join to un-aggregated child (fan-out duplicates parent rows) |
| `APPLY`/`LATERAL` for "latest N per parent" | Window function would read the table once and be cheaper at large N |
| CTE for readability | Assuming a CTE caches results |

### 9. Common mistakes
- Non-aggregate scalar subquery that can return two rows → runtime error in production → aggregate or `TOP 1 ORDER BY`, or enforce uniqueness.
- `NOT IN` with a nullable column → empty result → `NOT EXISTS`.
- Unqualified column inside subquery binds to the inner table → condition always true → always alias and qualify.
- Expecting a missing match to give `0` → get `NULL` and propagate it → `COALESCE(( … ), 0)`.
- Writing `WHERE x = (SELECT …)` where the subquery returns `NULL` → no rows → `IS NOT DISTINCT FROM` or handle `NULL`.
- 8,000 literals in an `IN` → plan cache bloat and slow compile → join to a temp table or `VALUES`.

### 10. 🎯 Interview kit
**30-second answer:** A scalar subquery returns one column, at most one row; zero rows becomes `NULL`, more than one is an error. It can be self-contained or correlated; correlated ones are logically evaluated per outer row, though the optimizer often turns them into joins. I prefer `EXISTS` over `IN` for correlated presence checks and `NOT EXISTS` over `NOT IN` because of `NULL`.

**Structure a longer answer:** define (scalar vs multi-value vs derived) → cardinality and NULL rules → correlated vs self-contained → rewrites (join, APPLY, window) → performance (index, plan operators) → pitfalls.

**Likely follow-ups:**
- *Difference between `IN` and `EXISTS`?* → semantics with NULLs; both usually become semi joins.
- *When is a correlated subquery slow?* → no index on the correlated column, large outer set, not decorrelated.
- *CTE vs temp table?* → CTE is just a name; temp table materializes and can have stats/indexes.
- *Why does `SELECT 1` work in `EXISTS`?* → select list ignored.

**Signals interviewers look for:** NULL awareness, cardinality reasoning, reading plans, choosing set-based rewrites.

### 11. Check yourself
- [ ] Predict the result of `NOT IN` when the subquery contains a `NULL`.
- [ ] Rewrite a correlated select-list subquery as a join with pre-aggregation.
- [ ] Explain what an Assert operator does and how to avoid it.
- [ ] Get the latest order per customer with `APPLY`/`LATERAL` and with `ROW_NUMBER()`.
- [ ] Replace a large literal `IN` list with a join to a temp table.
<!-- explainer:end -->

## Implementations (tools)

- [SQL Server](<../../../TOOLS/SQL_Server/_index.md>)
- [MySQL](<../../../TOOLS/MySQL/_index.md>)

## Practice questions

- [Basic] [What are the main sublanguages of SQL and which category does SELECT belong to?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39758_What_are_the_main_sublanguages_of_SQL_and_which_category_doe.md>)
- [Basic] [What is a table alias and why is it almost mandatory when joining a table to itself?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39830_What_is_a_table_alias_and_why_is_it_almost_mandatory_when_jo.md>)
- [Basic] [How do you sort query results by two columns, one ascending and one descending?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39922_How_do_you_sort_query_results_by_two_columns_one_ascending_a.md>)
- [Intermediate] [What does SELECT 1 FROM a table return, and why do people sometimes write it inside an EXISTS check?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39772_What_does_SELECT_1_FROM_a_table_return_and_why_do_people_som.md>)
- [Intermediate] [What is the difference between a scalar subquery and a subquery that returns multiple rows?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39840_What_is_the_difference_between_a_scalar_subquery_and_a_subqu.md>)
- [Intermediate] [Explain the difference between DISTINCT ON and ROW_NUMBER() = 1 for picking the latest row per group, and what ORDER BY guarantees you need for determinism.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39907_Explain_the_difference_between_DISTINCT_ON_and_ROW_NUMBER_1_.md>)
- [Intermediate] [How would you use SQL to sample 100 random conversations for a manual quality review, and why avoid ORDER BY RANDOM on a huge table?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40792_How_would_you_use_SQL_to_sample_100_random_conversations_for.md>)
- [Deep] [Why does a query filtering WHERE status IN (list of 8000 app-generated literals) produce an awful plan, and what rewrites (VALUES join, temp table, unnest array) stabilize it?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39869_Why_does_a_query_filtering_WHERE_status_IN_list_of_8000_app-.md>)

## Related concepts

- [WHERE vs HAVING](<../../Database_Internals/DBMS_Concepts/WHERE_vs_HAVING.md>) *(DBMS Concepts)*
- [OR Conditions and Index Usage](<../../../TOOLS/SQL_Server/SQL_Server_Platform/OR_Conditions_and_Index_Usage.md>) *(SQL Server Platform)*
- [UDF Performance](<../SQL_Functions/UDF_Performance.md>) *(SQL Functions)*

## References

- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Single-table queries
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Single-table queries (cont.)
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Table expressions
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Set operators
- *Grokking Relational Database Design* — Qiang Hao, Michail Tsikerdekis — Grokking Relational Database Design
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Writing Simple SELECT Queries › Generating a SELECT List
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Writing Simple SELECT Queries › Filtering Data
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Writing Simple SELECT Queries › Using WHERE Clauses with Two Predicates
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Writing Simple SELECT Queries › Using the IN Operator
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › READ (SELECT)
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Role of the WHERE Clause
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Practical Examples of WHERE Logic
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Conclusion
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Spark SQL and DataFrames Interacting with External Data Sources › Option 1: Explode and Collect
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — T-SQL development › 5.3 #16 Using SELECT \ as standard
- *SQL Server Tutorial* (YouTube series) — Part 130: Offset fetch next in SQL Server 2012 ([watch](https://www.youtube.com/watch?v=7JINCqgT_cQ))

