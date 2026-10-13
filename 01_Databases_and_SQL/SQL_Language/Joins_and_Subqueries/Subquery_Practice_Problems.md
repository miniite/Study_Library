---
id: "01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Subquery_Practice_Problems"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries"
tools: []
---

# Subquery Practice Problems

**L4 Advanced** · [↑ Joins and Subqueries](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A set of worked subquery problems — above-average filters, per-group maxima, "top per category", most-frequent customer, derived-table totals — showing which subquery shape (scalar, correlated, `IN`, derived table) fits each and how to check for ties and NULLs.

### 1. The big picture
Subquery problems come in a small number of shapes. Once you can name the shape you can write the query in one pass: *compare to a global value*, *compare to a group value*, *find the extreme row*, *top-N per group*, *aggregate then filter the aggregate*, *membership/absence*. This lesson practises the shapes with a recurring `employees`, `departments`, `orders`, `bookings` schema and annotates the usual traps (ties, NULLs, grain).

### 2. Core concepts and vocabulary
- **Self-contained scalar subquery** — one value, evaluated once (`> (SELECT AVG(…) …)`).
- **Correlated subquery** — depends on outer row (per-group comparisons).
- **Derived table** — subquery in `FROM`, aliased, used to aggregate first and filter afterwards.
- **Extremum with ties** — "the highest" may be several rows; `= (SELECT MAX …)` returns all ties, `TOP 1` returns one arbitrarily.
- **Top-N per group** — largest N rows per category; needs correlation or `ROW_NUMBER`/`RANK`.
- **Nested correlated subquery** — a correlated subquery that contains another correlated reference to the outer query (e.g. above department average but below department maximum).

### 3. How it works, step by step
For each problem:
1. Restate in one sentence: *compare what to what*.
2. Is the comparison value global, per group, or per row?
3. Write the inner query first and run it alone (self-contained subqueries are easy to debug).
4. Plug it in and check row counts, ties and NULLs.

### 4. Worked example
```sql
-- P1. Employees earning more than the company average (global value, scalar subquery)
SELECT name, salary FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- P2. Above their own department's average (per-group, correlated)
SELECT e.name, e.salary
FROM employees e
WHERE e.salary > (SELECT AVG(x.salary) FROM employees x WHERE x.dept_id = e.dept_id);

-- P3. Departments whose average salary exceeds the company average (aggregate then compare)
SELECT dept_id, AVG(salary) AS avg_sal
FROM employees
GROUP BY dept_id
HAVING AVG(salary) > (SELECT AVG(salary) FROM employees);

-- P4. Customers with the most bookings (extremum, ties preserved)
SELECT customer_id, COUNT(*) AS n
FROM bookings
GROUP BY customer_id
HAVING COUNT(*) = (SELECT MAX(cnt) FROM (SELECT COUNT(*) AS cnt FROM bookings GROUP BY customer_id) t);

-- P5. Top product by sales in each category (correlated max)
SELECT p.category_id, p.product_id, s.total
FROM (SELECT product_id, SUM(amount) AS total FROM sales GROUP BY product_id) s
JOIN products p ON p.product_id = s.product_id
WHERE s.total = (SELECT MAX(s2.total)
                 FROM (SELECT product_id, SUM(amount) AS total FROM sales GROUP BY product_id) s2
                 JOIN products p2 ON p2.product_id = s2.product_id
                 WHERE p2.category_id = p.category_id);
-- Simpler and one pass: RANK() OVER (PARTITION BY category_id ORDER BY total DESC) = 1

-- P6. Per-customer totals, then keep those above 10000 (derived table)
SELECT t.cust_id, t.total
FROM (SELECT cust_id, SUM(amount) AS total FROM orders GROUP BY cust_id) AS t
WHERE t.total > 10000;

-- P7. Employee with each employee's department average as a column (subquery in SELECT)
SELECT e.name, e.salary,
       (SELECT AVG(x.salary) FROM employees x WHERE x.dept_id = e.dept_id) AS dept_avg
FROM employees e;

-- P8. Managers and the count of direct reports (self-join + aggregate)
SELECT m.name AS manager, COUNT(e.emp_id) AS reports
FROM employees m JOIN employees e ON e.manager_id = m.emp_id
GROUP BY m.emp_id, m.name;

-- P9. Above department average but below department maximum (nested correlated)
SELECT e.name
FROM employees e
WHERE e.salary > (SELECT AVG(e2.salary) FROM employees e2 WHERE e2.dept_id = e.dept_id)
  AND e.salary < (SELECT MAX(e3.salary) FROM employees e3 WHERE e3.dept_id = e.dept_id);
```
Notes: P3 – a derived table also works; the `HAVING` form avoids it. P4 – `TOP 1 WITH TIES` is an alternative in SQL Server. P5 – the window version is the better answer in interviews. P9 – the two predicates can be combined into one derived table with `AVG` and `MAX` per department.

### 5. Under the hood   *(optional at L4)*
- P1: the scalar subquery has no correlation, so the plan computes it once (a *Stream Aggregate* feeding a *Nested Loops* with the main scan).
- P2/P9: SQL Server turns correlated aggregates into a join with a grouped aggregate (one pass) if it can; check for a *Spool* or repeated *Index Seek* to see if it didn't.
- P4: two aggregation levels — the inner `GROUP BY` produces counts, the outer picks the maximum; a window function `MAX(COUNT(*)) OVER ()` can do it in one pass.
- NULLs: `AVG` ignores NULL salaries; comparisons with NULL are UNKNOWN so such rows are excluded from results silently.

### 6. Performance and design implications
- A correlated subquery with an index on `(dept_id) INCLUDE (salary)` is cheap even per row; without it each execution scans the table.
- Aggregating once in a derived table and joining beats repeating the same aggregate in several correlated subqueries (P9).
- Use windows for "top N per group".
- Always test both ties and empty groups.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `> (SELECT AVG(…))` | Compare with a global scalar | Above-average filters |
| `= (SELECT MAX(…))` | Extremum with ties | "the highest" |
| `TOP 1 WITH TIES` | Extremum with ties (T-SQL) | Short queries |
| `ROW_NUMBER` / `RANK` / `DENSE_RANK` | Ranking per group | Top-N per group |
| `HAVING … (subquery)` | Filter groups by a scalar | Group vs global |
| Derived table in `FROM` | Aggregate then filter | Two-stage logic |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Scalar subquery | The value is needed in output too — compute once in a derived table |
| Correlated subquery | Large outer sets without an index on the correlation column |
| `RANK` vs `ROW_NUMBER` | Ties matter: `RANK` keeps them, `ROW_NUMBER` picks arbitrarily |
| `HAVING` with a subquery | The subquery is correlated to a group — evaluated per group |

### 9. Common mistakes
- `=` against a subquery that can return several rows → error 512 → `IN`, `MAX`, or fix the key.
- `salary > ALL (SELECT salary …)` with NULLs → no rows → filter NULLs.
- "Highest" with `ORDER BY … TOP 1` when ties exist → silently drops tied rows.
- Aliasing missing on a derived table → syntax error.
- Aggregating at the wrong grain (counting after a join) → see *Counting with Joins*.

### 10. 🎯 Interview kit
**30-second answer:** "I identify whether the comparison value is global, per group or per row. Global → scalar subquery, per group → correlated subquery or a window, aggregate-then-filter → derived table or `HAVING`. I handle ties with `=`/`RANK` and NULLs explicitly."

**Structure a longer answer:** shape → inner query alone → combine → ties/NULLs → optimize with window or derived table.

**Likely follow-ups:**
- *Why is the window version better?* → single scan, no repeated correlated execution, ties via `RANK`.
- *Departments above the company average?* → `HAVING AVG(salary) > (SELECT AVG(salary) …)`.
- *What if the subquery returns no row?* → scalar subquery returns NULL → predicate UNKNOWN → no rows.

**Signals interviewers look for:** correct handling of ties, empty results and NULLs; ability to rewrite.

### 11. Check yourself
- [ ] Solve P1–P9 on a blank page; verify against a sample dataset.
- [ ] Rewrite P2, P5 and P9 using window functions.
- [ ] Add a department with no employees and see what each query returns.
- [ ] Add NULL salaries and predict the effect on P1 and P2.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [Write a query using a subquery to find employees who earn more than the average salary.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30396_Write_a_query_using_a_subquery_to_find_employees_who_earn_mo.md>)
- [Intermediate] [Write a query to get each student's name and their highest exam score using a join.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30454_Write_a_query_to_get_each_student_s_name_and_their_highest_e.md>)
- [Intermediate] [Explain how you would list all managers and the count of people reporting directly to them.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30465_Explain_how_you_would_list_all_managers_and_the_count_of_peo.md>)
- [Deep] [How can a subquery in the SELECT list add a computed value like each employee's department average salary?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30419_How_can_a_subquery_in_the_SELECT_list_add_a_computed_value_l.md>)
- [Deep] [Write a query to find the top product by sales in each category using a subquery.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30432_Write_a_query_to_find_the_top_product_by_sales_in_each_categ.md>)
- [Deep] [Write a query to list departments whose average salary is above the company-wide average salary.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30440_Write_a_query_to_list_departments_whose_average_salary_is_ab.md>)
- [Deep] [Given a Bookings table write a subquery-based query to find the customer who made the most bookings.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30459_Given_a_Bookings_table_write_a_subquery-based_query_to_find_.md>)
- [Deep] [Write a query using a subquery in FROM to first compute per-customer totals then filter those above 10000.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30463_Write_a_query_using_a_subquery_in_FROM_to_first_compute_per-.md>)

## Related concepts

- [Basic Query Practice](<../../../TOOLS/MySQL/MySQL_Platform/Basic_Query_Practice.md>) *(MySQL Platform)*
- [Aggregation Query Practice](<../SQL_Fundamentals/Aggregation_Query_Practice.md>) *(SQL Fundamentals)*
- [Top-N per Group](<../Window_Functions/Top-N_per_Group.md>) *(Window Functions)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › Advanced Examples
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Writing Simple SELECT Queries › Filtering on Date and Time

