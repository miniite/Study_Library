---
id: "TOOLS/MySQL/MySQL_Platform/Basic_Query_Practice"
level: 4
type: "concept"
topic: "TOOLS/MySQL/MySQL_Platform"
tools: ["MySQL"]
---

# Basic Query Practice

**L4 Advanced** · [↑ MySQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Most everyday SQL is a handful of patterns — filter, de-duplicate, aggregate with GROUP BY/HAVING, join (including outer joins for zero counts), self-join and subquery for "latest per group" — and once you know the logical order in which clauses are evaluated, you can write and debug all of them.

### 1. The big picture
Basic query practice is where SQL stops being syntax and becomes habit. The same ten shapes cover most reporting needs: *list/filter*, *distinct*, *range*, *group and count*, *filter the groups*, *join*, *left join to keep empties*, *self-join*, *correlated subquery*, and *top-N per group*. This lesson works through them in MySQL 8 syntax; the logic is the same in T-SQL and PostgreSQL, with small differences in `LIMIT` vs `TOP`.

### 2. Core concepts and vocabulary
- **Predicate** — a condition in `WHERE`/`ON`/`HAVING` that evaluates to TRUE, FALSE or UNKNOWN.
- **NULL / three-valued logic** — comparisons with NULL give UNKNOWN; rows are kept only if the predicate is TRUE.
- **`DISTINCT`** — removes duplicate result rows.
- **`GROUP BY`** — collapses rows into groups; every selected non-aggregated column must be grouped.
- **Aggregate** — `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`.
- **`WHERE` vs `HAVING`** — filters rows before grouping vs filters groups after aggregation.
- **Inner / left outer join** — matched rows only vs all left rows with NULLs where unmatched.
- **Self-join** — a table joined to itself via aliases.
- **Correlated subquery** — an inner query that references the outer row.
- **Window function** — computes over a set of rows without collapsing them (`ROW_NUMBER() OVER (…)`).

### 3. How it works, step by step — logical processing order
1. `FROM` / `JOIN` — build the row set.
2. `WHERE` — filter rows.
3. `GROUP BY` — form groups.
4. `HAVING` — filter groups.
5. `SELECT` — compute expressions and aliases; `DISTINCT` is applied.
6. `ORDER BY` — sort (can use aliases).
7. `LIMIT` — take the first rows.

This is why `WHERE total > 5` can't use an alias or aggregate defined in `SELECT`, and why `HAVING COUNT(*) > 5` works.

### 4. Worked example
Tables: `customers(customer_id, name, city, manager_id)`, `orders(order_id, customer_id, order_date, amount)`, `products(product_id, name, category, price)`.

```sql
-- 1. Distinct cities
SELECT DISTINCT city FROM customers;

-- 2. Products between 100 and 500 (inclusive)
SELECT name, price FROM products WHERE price BETWEEN 100 AND 500;

-- 3. Customers with more than five orders
SELECT customer_id, COUNT(*) AS orders_cnt
FROM orders
GROUP BY customer_id
HAVING COUNT(*) > 5;

-- 4. Customers with their order counts (keep customers with none)
SELECT c.customer_id, c.name, COUNT(o.order_id) AS orders_cnt
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.customer_id
GROUP BY c.customer_id, c.name;

-- 5. Categories with counts, including empty categories
SELECT cat.category, COUNT(p.product_id) AS n
FROM (SELECT DISTINCT category FROM products) cat      -- or a categories table
LEFT JOIN products p ON p.category = cat.category
GROUP BY cat.category;

-- 6. Most recent order per customer: correlated subquery
SELECT o.*
FROM orders o
WHERE o.order_date = (SELECT MAX(o2.order_date)
                      FROM orders o2
                      WHERE o2.customer_id = o.customer_id);

-- 6b. Same with a window function (MySQL 8+; handles ties deterministically)
SELECT * FROM (
  SELECT o.*, ROW_NUMBER() OVER (PARTITION BY customer_id
                                 ORDER BY order_date DESC, order_id DESC) rn
  FROM orders o) t
WHERE rn = 1;

-- 7. Employees sharing a manager (self-join)
SELECT a.name AS emp1, b.name AS emp2, a.manager_id
FROM customers a
JOIN customers b ON a.manager_id = b.manager_id AND a.customer_id < b.customer_id;

-- 8. Departments with more than 10 employees
SELECT department_id, COUNT(*) AS headcount
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 10;
```
Read the results by asking: how many rows should come out (one per customer? one per order?) and does a join multiply rows? `a.customer_id < b.customer_id` in the self-join removes mirrored pairs and self-pairs.

### 5. Under the hood
- **LEFT JOIN + COUNT.** `COUNT(*)` counts the NULL-extended row as 1, so a customer with no orders shows 1; `COUNT(o.order_id)` ignores NULLs and gives 0.
- **Filtering the right table of a LEFT JOIN** in `WHERE` (`WHERE o.status = 'paid'`) turns it into an inner join, because NULL ≠ 'paid'. Put the condition in `ON` to keep unmatched left rows.
- **Correlated subquery** executes conceptually once per outer row; the optimizer may turn it into a semi-join or use an index on `(customer_id, order_date)`. For big tables, a window function or a join to a grouped derived table is usually clearer and faster.
- **Ties.** `= MAX(date)` returns *all* tied rows; use `ROW_NUMBER()` with a tie-breaker to return exactly one.
- **`DISTINCT` vs `GROUP BY`.** Equivalent for plain de-duplication; `GROUP BY` is needed when aggregating.
- **`NOT IN` with NULLs** returns no rows if the subquery yields a NULL; use `NOT EXISTS`.
- **MySQL specifics**: `ONLY_FULL_GROUP_BY` (default on) rejects ungrouped select columns; `LIMIT n OFFSET m` for pagination; string comparison is case-insensitive under default collations; integer division with `/` yields decimals (use `DIV` for integer).

### 6. Performance and design implications
- Index the columns used in `WHERE`, `JOIN ON` and `GROUP BY`; composite indexes should list equality columns first.
- Avoid functions on indexed columns in predicates (`WHERE YEAR(order_date) = 2025` → `order_date >= '2025-01-01' AND order_date < '2026-01-01'`).
- Select only the columns you need; `SELECT *` defeats covering indexes.
- `EXPLAIN` shows join order and index choice: look for `type = ALL` (full scan) on big tables and `Using temporary; Using filesort` for large groupings.
- Aggregate *before* joining when the other side is large to avoid multiplying rows.
- Test on realistic row counts; patterns that work on 100 rows may collapse on 10 million.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SELECT DISTINCT` | Remove duplicate rows | Unique lists |
| `GROUP BY … HAVING` | Aggregate and filter groups | Counts, thresholds |
| `LEFT JOIN … ON` | Keep unmatched left rows | Zero-count reports |
| `ROW_NUMBER()/RANK()` | Rank within group | Top-N per group |
| `EXISTS` / `NOT EXISTS` | Semi/anti-join | Presence tests, NULL-safe |
| `EXPLAIN [ANALYZE]` | Plan | Tuning |
| `LIMIT … OFFSET` | Row limit | Pagination (keyset for deep pages) |
| `sql_mode` (`ONLY_FULL_GROUP_BY`) | Strictness | Catch ambiguous groupings |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Window function for top-N per group | Very old MySQL (< 8.0) without window support |
| Correlated subquery | Large tables without a supporting index |
| `EXISTS` | Need columns from the other table (use a join) |
| `JOIN` + `GROUP BY` | Join multiplies rows before aggregation → aggregate first |
| `DISTINCT` | Masking a join duplication bug |

### 9. Common mistakes
- Selecting non-grouped columns → error or arbitrary value → group or aggregate them.
- `WHERE COUNT(*) > 5` → invalid → use `HAVING`.
- Filtering outer-joined table in `WHERE` → silently inner join → move predicate to `ON`.
- `COUNT(*)` after a left join → zero counted as 1 → count a column from the right table.
- Using `= NULL` → never true → `IS NULL`.
- `DISTINCT` to hide duplicates → wrong totals → fix the join key.

### 10. 🎯 Interview kit
**30-second answer:** SQL evaluates FROM/JOIN, WHERE, GROUP BY, HAVING, SELECT, ORDER BY, LIMIT. I filter rows in WHERE, aggregate in GROUP BY, filter groups with HAVING, use LEFT JOIN with COUNT(column) for zero counts, and use ROW_NUMBER or a correlated subquery for latest-per-group.

**Structure a longer answer:** state the grain of the result → pick the join type → filter before or after aggregation → handle ties/NULLs → mention indexes and plan.

**Likely follow-ups:**
- *WHERE vs HAVING?* → rows vs groups.
- *Why did my left join lose rows?* → predicate on right table in WHERE.
- *Latest order per customer without window functions?* → correlated subquery or join to `GROUP BY` derived table.
- *How would you speed it up?* → composite index `(customer_id, order_date)`.

**Signals interviewers look for:** clear grain, NULL awareness, correct join semantics, index sense.

### 11. Check yourself
- [ ] Write the logical processing order from memory.
- [ ] Produce customers with zero orders in one query.
- [ ] Return exactly one latest order per customer, even with ties.
- [ ] Explain why `NOT IN` can return no rows.
- [ ] Rewrite `WHERE YEAR(order_date)=2025` to be index-friendly.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../_index.md>)

## Practice questions

- [Basic] [How would you write a query that returns distinct city names from a customers table?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33351_How_would_you_write_a_query_that_returns_distinct_city_names.md>)
- [Basic] [How would you write a query to find all products priced between 100 and 500?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33375_How_would_you_write_a_query_to_find_all_products_priced_betw.md>)
- [Basic] [Write a query using GROUP BY to count the number of orders per customer.](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33069_Write_a_query_using_GROUP_BY_to_count_the_number_of_orders_p.md>)
- [Intermediate] [How would you list departments that have more than 10 employees using GROUP BY and HAVING?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33400_How_would_you_list_departments_that_have_more_than_10_employ.md>)
- [Intermediate] [How would you join a customers table and an orders table to show each customer with their order count?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33403_How_would_you_join_a_customers_table_and_an_orders_table_to_.md>)
- [Intermediate] [How would you show all customers even if they have no orders, and what join achieves that?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33410_How_would_you_show_all_customers_even_if_they_have_no_orders.md>)
- [Intermediate] [How do you decide between GROUP BY with MAX and a self-join to fetch the row with the latest timestamp per group, and what does each cost?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33488_How_do_you_decide_between_GROUP_BY_with_MAX_and_a_self-join_.md>)
- [Intermediate] [Write a query to find employees earning more than the average salary of their own department.](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33030_Write_a_query_to_find_employees_earning_more_than_the_averag.md>)

## Related concepts

- [Aggregation Query Practice](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Aggregation_Query_Practice.md>) *(SQL Fundamentals)*
- [Classic SQL Query Problems](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Classic_SQL_Query_Problems.md>) *(DBMS Concepts)*
- [Subquery Practice Problems](<../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Subquery_Practice_Problems.md>) *(Joins and Subqueries)*

## References

- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Single-table queries (cont.)
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Subqueries
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Pagination with the TOP Clause
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Understanding the OVER Clause
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Joining Tables › Joining Three or More Tables

