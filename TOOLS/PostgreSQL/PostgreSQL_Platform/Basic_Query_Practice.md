---
id: "TOOLS/PostgreSQL/PostgreSQL_Platform/Basic_Query_Practice"
level: 4
type: "concept"
topic: "TOOLS/PostgreSQL/PostgreSQL_Platform"
tools: []
---

# Basic Query Practice

**L4 Advanced** · [↑ PostgreSQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Almost every everyday reporting question is one of six shapes — filter, aggregate per group, filter groups, "who has none", "top N per group", and paging — and each has a standard PostgreSQL form you should be able to write without thinking.

### 1. The big picture
Basic query practice is about pattern recognition. A question like "customers with more than five orders" or "the third page of results" maps to a known skeleton: `WHERE` for rows, `GROUP BY` + `HAVING` for groups, `LEFT JOIN … IS NULL` or `NOT EXISTS` for absence, a window function for "per group, ranked", and `LIMIT/OFFSET` (or keyset) for paging. Getting the logical order of evaluation right is the whole game: a query is *written* `SELECT … FROM … WHERE …` but *evaluated* `FROM → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT`. Most beginner bugs (using an alias in `WHERE`, filtering an aggregate in `WHERE`) are evaluation-order bugs.

This lesson builds on PostgreSQL query basics and prepares you for joins, window functions and performance tuning.

### 2. Core concepts and vocabulary
- **Predicate** — a condition that is `TRUE`, `FALSE` or `NULL` (unknown) for each row; `WHERE` keeps only `TRUE` rows.
- **Aggregate** — collapses many rows to one value (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`).
- **`GROUP BY`** — partitions rows into groups; every selected column must be grouped or aggregated.
- **`HAVING`** — a filter applied to groups after aggregation (`WHERE` runs before it).
- **Anti-join** — "rows in A with no match in B", written `LEFT JOIN … WHERE b.id IS NULL` or `NOT EXISTS`.
- **Window function** — computes over a set of related rows *without* collapsing them (`ROW_NUMBER() OVER (PARTITION BY …)`).
- **Three-valued logic** — comparisons with `NULL` yield `NULL`, not `FALSE`; `NULL = NULL` is not true.
- **`FILTER (WHERE …)`** — PostgreSQL clause that restricts which rows feed one aggregate.

### 3. How it works, step by step
Logical processing order of one `SELECT`:

1. `FROM` / `JOIN` — build the row set.
2. `WHERE` — drop rows whose predicate isn't `TRUE`.
3. `GROUP BY` — form groups.
4. `HAVING` — drop groups.
5. `SELECT` — compute expressions and aliases; window functions are computed here, *after* grouping.
6. `DISTINCT`, then `ORDER BY` (aliases are allowed here), then `LIMIT/OFFSET`.

Consequences: you can't reference a `SELECT` alias in `WHERE` or `HAVING`; you can't put `COUNT(*) > 5` in `WHERE`; window functions can't appear in `WHERE` (wrap in a subquery or CTE and filter outside).

### 4. Worked example
Schema: `customers(customer_id, name, is_premium)`, `orders(order_id, customer_id, order_date, amount)`, `employees(emp_id, dept, salary)`.

```sql
-- 1. Range filter. BETWEEN is inclusive on both ends -> bad for timestamps.
SELECT * FROM orders
WHERE order_date >= DATE '2026-01-01' AND order_date < DATE '2026-02-01';

-- 2. Several counts in one pass
SELECT COUNT(*) FILTER (WHERE is_premium)     AS premium,
       COUNT(*) FILTER (WHERE NOT is_premium) AS regular
FROM customers;

-- 3. Customers with more than five orders
SELECT customer_id, COUNT(*) AS n
FROM orders GROUP BY customer_id HAVING COUNT(*) > 5;

-- 4. Customers with no orders (anti-join)
SELECT c.* FROM customers c
WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id);

-- 5. Average orders per customer (include zero-order customers!)
SELECT AVG(n) FROM (
  SELECT c.customer_id, COUNT(o.order_id) AS n
  FROM customers c LEFT JOIN orders o USING (customer_id)
  GROUP BY c.customer_id) t;

-- 6. Top 3 salaries per department
SELECT * FROM (
  SELECT dept, emp_id, salary,
         DENSE_RANK() OVER (PARTITION BY dept ORDER BY salary DESC) AS rnk
  FROM employees) t
WHERE rnk <= 3;

-- 7. Page 3, 20 rows per page
SELECT * FROM orders ORDER BY order_id LIMIT 20 OFFSET 40;
```

Reading them: query 5 uses `COUNT(o.order_id)` rather than `COUNT(*)` — with a left join, a customer with no orders still yields one row (with `NULL` order columns), so `COUNT(*)` would report 1 instead of 0. Query 6 uses `DENSE_RANK` so ties share a rank; `ROW_NUMBER` would pick an arbitrary winner among ties, and `RANK` would leave gaps.

### 5. Under the hood   *(optional at this level)*
- `GROUP BY` is executed by `HashAggregate` (hash table of groups) or `GroupAggregate` (sort, then stream). `HAVING` conditions that don't involve aggregates are pushed down into `WHERE` by the planner, but write them in `WHERE` yourself.
- `NOT EXISTS` becomes a *hash anti-join* or *merge anti-join*; `NOT IN (subquery)` often cannot, because of `NULL` semantics (see mistakes).
- `OFFSET n` must still *produce and discard* n rows, so deep pages are linear in n.

### 6. Performance and design implications
- Always give `LIMIT` an `ORDER BY` on a unique tiebreaker, or pages overlap/skip.
- Prefer **keyset pagination** (`WHERE (created_at, id) < ($1, $2) ORDER BY created_at DESC, id DESC LIMIT 20`) for big tables: constant cost per page with a matching index.
- Filter early: put row filters in `WHERE`, not `HAVING`, so fewer rows are aggregated.
- Use `EXPLAIN (ANALYZE, BUFFERS)` to confirm an index is used rather than guessing.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `COUNT(*) FILTER (WHERE …)` | conditional aggregate | several counts in one scan |
| `LIMIT n OFFSET m` | row window | small, shallow pages |
| `FETCH FIRST n ROWS ONLY` | standard-SQL equivalent of `LIMIT` | portable code |
| `DISTINCT ON (col)` | first row per group (PG-only) | "latest order per customer" |
| `COALESCE(x, 0)` | replace `NULL` | null-safe arithmetic |
| `EXPLAIN (ANALYZE, BUFFERS)` | actual plan + I/O | verifying any query |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `NOT EXISTS` for "has none" | `NOT IN` over a nullable column (one `NULL` makes the result empty) |
| `LEFT JOIN … IS NULL` when you also need joined columns | joins that multiply rows before aggregating |
| Window function for top-N per group | correlated subquery per group on large data |
| `DISTINCT ON` for "latest per key" | when you need ties or strict SQL portability |
| Keyset pagination | users need arbitrary "jump to page 500" |

### 9. Common mistakes
- `WHERE COUNT(*) > 5` → error "aggregate functions are not allowed in WHERE" → use `HAVING`.
- `BETWEEN '2026-01-01' AND '2026-01-31'` on a timestamp → misses most of Jan 31 → use half-open ranges `>= … AND < next day`.
- `= NULL` → never true → use `IS NULL` / `IS DISTINCT FROM`.
- `NOT IN (SELECT col …)` with a `NULL` in the subquery → zero rows → use `NOT EXISTS`.
- Selecting a non-grouped column → error → add it to `GROUP BY` or aggregate it.
- `OFFSET` pagination without a deterministic `ORDER BY` → duplicates between pages → add a unique tiebreaker.

### 10. 🎯 Interview kit
**30-second answer:** "Evaluation order is `FROM`, `WHERE`, `GROUP BY`, `HAVING`, `SELECT`, `ORDER BY`, `LIMIT`. Row filters go in `WHERE`, group filters in `HAVING`; absence is `NOT EXISTS`; top-N per group is a window function; big pagination is keyset."
**Structure a longer answer:** clarify the grain of the table → pick the skeleton → handle `NULL`/ties/empty groups → mention the index that supports it.
**Likely follow-ups:** *`RANK` vs `DENSE_RANK` vs `ROW_NUMBER`?* → gaps vs no gaps vs arbitrary unique. *`WHERE` vs `HAVING`?* → before vs after grouping. *Why is `OFFSET` slow?* → it reads and discards rows.
**Signals interviewers look for:** you ask about duplicates and `NULL`s, you use half-open date ranges, you state the evaluation order unprompted.

### 11. Check yourself
- [ ] Write "customers with no orders" three ways and say which one breaks with `NULL`s.
- [ ] Explain why `COUNT(*)` is wrong after a `LEFT JOIN` when you want zero counts.
- [ ] Rewrite an `OFFSET 100000` query as keyset pagination and name the index it needs.
- [ ] Produce the second-highest salary per department with and without a window function.
<!-- explainer:end -->

## Practice questions

- [Basic] [How do you fetch only the first 10 rows of a query result in PostgreSQL?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36147_How_do_you_fetch_only_the_first_10_rows_of_a_query_result_in.md>)
- [Intermediate] [How would you find employees who share the same salary using GROUP BY and HAVING?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36166_How_would_you_find_employees_who_share_the_same_salary_using.md>)
- [Intermediate] [How would you write a query to find products that have never been ordered?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36181_How_would_you_write_a_query_to_find_products_that_have_never.md>)
- [Intermediate] [How would you write a query to find gaps in a sequence of order IDs?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36191_How_would_you_write_a_query_to_find_gaps_in_a_sequence_of_or.md>)
- [Intermediate] [How would you write a query to find duplicate email addresses in a users table?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36199_How_would_you_write_a_query_to_find_duplicate_email_addresse.md>)
- [Intermediate] [How would you find courses that have no students enrolled?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36221_How_would_you_find_courses_that_have_no_students_enrolled.md>)
- [Intermediate] [Write a query to find all customers who have never placed an order using a LEFT JOIN or NOT EXISTS.](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35845_Write_a_query_to_find_all_customers_who_have_never_placed_an.md>)
- [Deep] [Write a query to compute a 7-day moving average of daily active users using a window function.](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35855_Write_a_query_to_compute_a_7-day_moving_average_of_daily_act.md>)

## Related concepts

- [Basic Query Practice](<../../MySQL/MySQL_Platform/Basic_Query_Practice.md>) *(MySQL Platform)*
- [Aggregation Query Practice](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Aggregation_Query_Practice.md>) *(SQL Fundamentals)*
- [Counting with Joins](<../../../01_Databases_and_SQL/SQL_Language/Aggregation/Counting_with_Joins.md>) *(Aggregation)*

## References

- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Single-table queries
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Infrastructure Operations › Query optimization

