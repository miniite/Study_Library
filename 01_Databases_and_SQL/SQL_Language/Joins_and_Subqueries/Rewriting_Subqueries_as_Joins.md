---
id: "01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Rewriting_Subqueries_as_Joins"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries"
tools: []
---

# Rewriting Subqueries as Joins

**L4 Advanced** · [↑ Joins and Subqueries](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Most subqueries have a join, derived-table, `EXISTS` or window-function equivalent; rewrite when it removes per-row re-evaluation or clarifies grain, but keep the subquery when the join would duplicate rows or change NULL semantics.

### 1. The big picture
A subquery is a query nested in another. The outer query is the *outer query*; the inner one supplies a value, a list of values, or a whole table. Subqueries are the natural way to say "salary above the average" or "customers who placed an order" without joining. But they are also the most common source of slow queries (correlated subqueries evaluated per row) and subtle bugs (accidental outer references, `NOT IN` with NULLs). Knowing the kinds of subquery, and how each can be rewritten, lets you pick the form that is both correct and fast.

### 2. Core concepts and vocabulary
- **Self-contained (non-correlated) subquery** — independent of the outer query; can be run and debugged alone; logically evaluated once.
- **Correlated subquery** — refers to a column of the outer query (`WHERE x.dept = e.dept`); logically re-evaluated per outer row.
- **Scalar subquery** — returns one value (one row, one column); usable wherever an expression is.
- **Multivalued subquery** — returns one column, many rows; used with `IN`, `ANY`, `ALL`, `EXISTS`.
- **Table subquery / derived table** — returns a relation; lives in `FROM`; needs an alias.
- **Subquery placement** — in `WHERE` (filter), `SELECT` (computed value per row), `FROM` (derived table), `HAVING`.
- **Decorrelation** — transforming a correlated subquery into a join/aggregate that is computed once.
- **Outer reference binding** — name resolution: an unqualified column inside a subquery that is missing from the inner table silently binds to the outer query.

### 3. How it works, step by step
**Rewrite catalogue**

| Subquery form | Rewrite | Notes |
|---|---|---|
| `WHERE id IN (SELECT id FROM t2)` | `JOIN t2` or `EXISTS` | Join may duplicate if `t2.id` not unique; `EXISTS` never does |
| `WHERE EXISTS (… correlated …)` | `JOIN` + `DISTINCT`, or leave | Usually already optimal as semi-join |
| `WHERE NOT IN (SELECT …)` | `NOT EXISTS` or `LEFT JOIN … IS NULL` | Fixes NULL trap |
| `WHERE sal > (SELECT AVG(sal) …)` | Window `AVG() OVER ()` or join to a one-row derived table | Self-contained: runs once anyway |
| Correlated scalar: `sal > (SELECT AVG(sal) … WHERE dept = e.dept)` | Join to `GROUP BY dept` derived table or `AVG() OVER (PARTITION BY dept)` | Removes per-row evaluation |
| Scalar in `SELECT` list: `(SELECT MAX(o.date) FROM orders o WHERE o.cust = c.cust)` | `LEFT JOIN` aggregated derived table or `OUTER APPLY` | Keeps customers without orders |
| Top-1 per group: `WHERE date = (SELECT MAX(date) …)` | `ROW_NUMBER() OVER (PARTITION BY …)` | Handles ties explicitly |

**Procedure**
1. Identify the subquery kind (self-contained vs correlated, scalar vs multivalued).
2. Decide the target form from the table above.
3. Check *grain*: will the join duplicate outer rows? If the inner side's key isn't unique, aggregate or use `EXISTS`.
4. Check *NULL semantics*: `NOT IN`/`=` comparisons.
5. Compare results (row counts, `EXCEPT` in both directions) and plans.

### 4. Worked example
```sql
-- Original: correlated scalar subquery in WHERE
SELECT e.name, e.salary, e.dept_id
FROM employees e
WHERE e.salary > (SELECT AVG(x.salary) FROM employees x WHERE x.dept_id = e.dept_id);

-- Rewrite 1: join to a derived table (aggregate computed once)
SELECT e.name, e.salary, e.dept_id
FROM employees e
JOIN (SELECT dept_id, AVG(salary) AS avg_sal
      FROM employees GROUP BY dept_id) d ON d.dept_id = e.dept_id
WHERE e.salary > d.avg_sal;

-- Rewrite 2: window function (single scan, keeps all columns available)
SELECT name, salary, dept_id
FROM (SELECT *, AVG(salary) OVER (PARTITION BY dept_id) AS avg_sal FROM employees) t
WHERE salary > avg_sal;

-- Subquery in IN -> semi-join, no duplicates
SELECT CustomerID, AccountNumber
FROM Sales.Customer c
WHERE EXISTS (SELECT 1 FROM Sales.SalesOrderHeader h WHERE h.CustomerID = c.CustomerID);

-- Hidden bug: outer-reference binding
SELECT c.cust_id
FROM customers c
WHERE c.cust_id IN (SELECT cust_id FROM orders_archive);   -- if orders_archive has NO cust_id column,
-- the name resolves to c.cust_id => condition is always true: every customer returned, no error.
-- Prevent: qualify columns (SELECT a.cust_id FROM orders_archive a).
```
Verify: `SELECT … EXCEPT SELECT …` in both directions should return zero rows when two versions are equivalent.

### 5. Under the hood   *(optional at L4)*
- SQL Server's optimizer performs **subquery unnesting**: `IN`/`EXISTS` become semi-joins, correlated aggregate subqueries often become a join with a group-by (an *Apply* converted to a join). If it cannot unnest (e.g. nested correlated subqueries with `TOP`, or the ones mixed with `OR`), you see *Nested Loops* with an *Index Spool* or repeated *Stream Aggregate*s: one execution per outer row.
- Derived tables are **inlined**; unless `TOP`/`OFFSET` forces it, they are not materialized. A "huge derived table" is only stored (spooled) if the plan shows a *Table Spool*; check the plan, not assumptions.
- A scalar subquery in the `SELECT` list is evaluated for each output row; with an index on the correlated column it is a seek per row (acceptable for hundreds of rows, expensive for millions).
- Correlated subqueries are not always worse: with a good index on the correlated key and a selective outer query, nested-loops plan can be fastest.

### 6. Performance and design implications
- Index the correlated column on the inner table.
- Prefer set-based rewrites (join/window) for large outer sets; keep correlated `EXISTS` for selective filters.
- Avoid correlated subqueries in `HAVING` and the select list on large sets.
- Use windows to avoid repeated scans of the same table (one scan instead of outer + inner).
- Readability: CTEs (`WITH`) can flatten deep nesting; they don't change semantics.
- Always compare plans and `STATISTICS IO`; equal-looking rewrites may produce identical plans.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `IN (subquery)` | Membership test | Simple, unique subquery columns |
| `EXISTS` / `NOT EXISTS` | Existence test, NULL-safe | Filters |
| `ANY` / `SOME` / `ALL` | Quantified comparison | `> ALL (SELECT …)` |
| Derived table `(SELECT …) AS d` | Inline relation | Pre-aggregation |
| `CROSS/OUTER APPLY` | Per-row table expression | Top-N per parent |
| Window functions | Aggregates without collapsing rows | Rewrites of correlated aggregates |
| `SET STATISTICS IO, TIME ON` | Cost comparison | Always |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Subquery for readability and self-contained logic | The same subquery text repeats; use a CTE |
| Join/derived rewrite for performance | The join changes row multiplicity |
| Window function | The engine is old/lacks windows; huge partitions spill |
| `EXISTS` | You need columns from the inner table |

### 9. Common mistakes
- Scalar subquery returns more than one row → runtime error 512 → aggregate or add a unique predicate.
- Unqualified column bound to the outer query → always-true filter → qualify names.
- Rewriting `IN` to `JOIN` on a non-unique key → duplicate rows → `EXISTS` or `DISTINCT`.
- `NOT IN` with NULLs → empty result → `NOT EXISTS`.
- Assuming nested correlated subqueries are cheap → quadratic → flatten via joins/windows.

### 10. 🎯 Interview kit
**30-second answer:** "Self-contained subqueries run once; correlated ones depend on the outer row and are logically per-row, though the optimizer usually unnests them. I rewrite `IN` as `EXISTS` or a join, correlated aggregates as a join to a grouped derived table or a window function, and I check duplicates and NULL semantics when I do."

**Structure a longer answer:** classify the subquery → pick a rewrite → grain and NULL checks → plan comparison.

**Likely follow-ups:**
- *Can a subquery be in FROM? What's it called?* → yes, a derived table; needs an alias; inlined by the optimizer.
- *CTE vs nested subquery?* → same semantics, CTE is more readable and may be referenced multiple times.
- *How does an accidental outer reference cause wrong results?* → missing inner column resolves to the outer one, making the predicate trivially true.
- *Subquery in HAVING?* → compares an aggregate against a computed value; correlated ones re-evaluate per group.

**Signals interviewers look for:** you know both correct and fast forms, mention NULL/duplicate pitfalls, and talk about plans.

### 11. Check yourself
- [ ] Rewrite "customers with an order" with `IN`, `EXISTS`, and `JOIN`, and say which can duplicate.
- [ ] Turn a correlated department-average filter into a window query.
- [ ] Find the bug where an unqualified column binds to the outer query.
- [ ] Show how to verify two rewrites are equivalent.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [Where in a query can a subquery appear (SELECT WHERE FROM HAVING)?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30395_Where_in_a_query_can_a_subquery_appear_SELECT_WHERE_FROM_HAV.md>)
- [Intermediate] [What is a derived table and do you need to give it an alias?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30409_What_is_a_derived_table_and_do_you_need_to_give_it_an_alias.md>)
- [Intermediate] [Explain the difference between correlated and non-correlated placement of a subquery in HAVING and how it affects grouping cost.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30329_Explain_the_difference_between_correlated_and_non-correlated.md>)
- [Deep] [How do you find duplicate rows in a table using a self join or subquery?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30430_How_do_you_find_duplicate_rows_in_a_table_using_a_self_join_.md>)
- [Deep] [Show me how you'd find the latest row per group and compare the window-function approach against a correlated subquery on cost.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30287_Show_me_how_you_d_find_the_latest_row_per_group_and_compare_.md>)
- [Deep] [You suspect a subquery is being evaluated per-partition instead of once — how do you read the plan to confirm and rewrite it?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30323_You_suspect_a_subquery_is_being_evaluated_per-partition_inst.md>)
- [Deep] [How do you use a windowed subquery (ROW_NUMBER in a CTE) to de-duplicate before a join, and why is it safer than DISTINCT?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30351_How_do_you_use_a_windowed_subquery_ROW_NUMBER_in_a_CTE_to_de.md>)
- [Deep] [Explain how a subquery used in an UPDATE ... FROM / correlated UPDATE can update the wrong rows if it returns multiple matches, and how to make it deterministic.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30373_Explain_how_a_subquery_used_in_an_UPDATE_FROM_correlated_UPD.md>)

## Related concepts

- [PostgreSQL Query Basics](<../../../TOOLS/PostgreSQL/PostgreSQL_Platform/PostgreSQL_Query_Basics.md>) *(PostgreSQL Platform)*
- [Scalar Subqueries](<../SQL_Fundamentals/Scalar_Subqueries.md>) *(SQL Fundamentals)*
- [CROSS APPLY and OUTER APPLY](<../../../TOOLS/SQL_Server/T-SQL/CROSS_APPLY_and_OUTER_APPLY.md>) *(T-SQL)*

## References

- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Subqueries
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › Correlated and Non-Correlated Subqueries
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › Non-Correlated Subqueries
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › Correlated Subqueries
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › Subqueries in Different Clauses
- *Hadoop: The Definitive Guide* — Tom White — Hive › Subqueries
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Building on Subqueries, Common Table Expressions, and Unions › Writing Subqueries
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Building on Subqueries, Common Table Expressions, and Unions › Using a Subquery and NOT IN
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Building on Subqueries, Common Table Expressions, and Unions › Using EXISTS
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Building on Subqueries, Common Table Expressions, and Unions › Using CROSS APPLY and OUTER APPLY
- *SQL Server Tutorial* (YouTube series) — Part 59: Subqueries in sql ([watch](https://www.youtube.com/watch?v=JtmfAGM4pfc))
- *SQL Server Tutorial* (YouTube series) — Part 60: Correlated subquery in sql ([watch](https://www.youtube.com/watch?v=Ra3ISwvcFlM))

