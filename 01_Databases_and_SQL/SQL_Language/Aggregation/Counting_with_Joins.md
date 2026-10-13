---
id: "01_Databases_and_SQL/SQL_Language/Aggregation/Counting_with_Joins"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/Aggregation"
tools: []
---

# Counting with Joins

**L4 Advanced** · [↑ Aggregation](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A join multiplies rows, so any `COUNT` or `SUM` computed *after* a one-to-many join counts the multiplied rows; either aggregate **before** joining, count `DISTINCT` keys, or use conditional aggregation (`SUM(CASE …)`) to get several counts in one pass.

### 1. The big picture
Aggregation questions in the real world almost always span more than one table: "orders per customer", "products bought per customer", "employees per manager", "pending vs shipped orders in one row". The moment a second table enters, the unit of the result changes. A customer with 3 orders and 5 order lines becomes 15 rows after joining both children. `COUNT(*)` then reports 15, not 3 and not 5. Most wrong numbers in reports come from this *fan-out*, and from using an inner join when customers with zero rows must appear with a count of 0.

This lesson sits under *Aggregation* (parent) and next to `COUNT_Semantics` and `GROUP_BY_and_HAVING`. It covers: counting per parent across a join, the inner-vs-outer join choice, `COUNT(DISTINCT …)` as a repair, pre-aggregation as the better repair, and conditional aggregation for pivot-style counts.

### 2. Core concepts and vocabulary
- **Fan-out (row multiplication)** — a one-to-many join returns the parent row once per matching child row, so parent-level measures are repeated.
- **Grain** — what one row of a result represents (one customer, one order, one order line). Wrong grain = wrong aggregate.
- **`COUNT(*)`** — counts rows in the group, including NULL-padded rows produced by outer joins.
- **`COUNT(col)`** — counts rows where `col` is not NULL; on the right side of a `LEFT JOIN` it counts only real matches.
- **`COUNT(DISTINCT col)`** — number of distinct non-NULL values; removes duplicates created by fan-out but not the duplicated *measure* in `SUM`.
- **Pre-aggregation** — aggregating a child table in a derived table/CTE to the parent's grain *before* joining.
- **Conditional aggregation** — an aggregate wrapping a `CASE` expression: `SUM(CASE WHEN cond THEN 1 ELSE 0 END)`.
- **Anti-join count** — customers with *no* matching rows; needs an outer join (or `NOT EXISTS`), not an inner join.

### 3. How it works, step by step
Take `customers(c)` → `orders(o)` → `order_items(i)` and the request "number of orders per customer".

1. **Logical order of evaluation** (the book's single-table chapter stresses this): `FROM/JOIN` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `ORDER BY`. The join is always completed *before* grouping, so groups are formed from joined rows.
2. After `customers JOIN orders` the grain is "one row per order": `COUNT(*)` per customer is right.
3. After also joining `order_items` the grain is "one row per order line": `COUNT(*)` per customer now means *lines*, and `COUNT(o.order_id)` also means lines (the order id repeats on each line).
4. Repairs, in order of preference:
   - **Pre-aggregate**: `(SELECT order_id, COUNT(*) n_items FROM order_items GROUP BY order_id)` then join at the order grain.
   - **`COUNT(DISTINCT o.order_id)`** when you only need counts, not sums.
   - **Avoid the second join** by using a correlated scalar subquery or `EXISTS` when you only need yes/no.
5. For customers with zero orders use `LEFT JOIN` and count a column from the **right** side (`COUNT(o.order_id)`), never `COUNT(*)` (which gives 1 for the NULL-padded row).

### 4. Worked example
```sql
-- Setup
CREATE TABLE customers (cust_id INT PRIMARY KEY, name VARCHAR(50));
CREATE TABLE orders    (order_id INT PRIMARY KEY, cust_id INT, ship_date DATE NULL, amount DECIMAL(10,2));
CREATE TABLE order_items(item_id INT PRIMARY KEY, order_id INT, product_id INT);

INSERT customers VALUES (1,'Ana'),(2,'Ben'),(3,'Cy');
INSERT orders VALUES (10,1,'2025-01-05',100),(11,1,NULL,50),(12,2,'2025-01-07',200);
INSERT order_items VALUES (1,10,7),(2,10,8),(3,10,7),(4,11,9),(5,12,7);

-- WRONG: fan-out, and Cy disappears
SELECT c.name, COUNT(*) AS orders_cnt, SUM(o.amount) AS revenue
FROM customers c
JOIN orders o       ON o.cust_id  = c.cust_id
JOIN order_items i  ON i.order_id = o.order_id
GROUP BY c.name;
-- Ana: 4 'orders', revenue 100+100+100+50 = 350 (true: 2 orders, 150). Cy missing.

-- RIGHT: pre-aggregate, then LEFT JOIN, count the right-side column
SELECT c.name,
       COUNT(o.order_id)               AS orders_cnt,
       COALESCE(SUM(o.amount), 0)      AS revenue,
       COALESCE(SUM(x.n_items), 0)     AS items
FROM customers c
LEFT JOIN orders o ON o.cust_id = c.cust_id
LEFT JOIN (SELECT order_id, COUNT(*) AS n_items
           FROM order_items GROUP BY order_id) x ON x.order_id = o.order_id
GROUP BY c.name;
-- Ana 2 / 150 / 4,  Ben 1 / 200 / 1,  Cy 0 / 0 / 0

-- Pending vs shipped in one pass (conditional aggregation)
SELECT SUM(CASE WHEN ship_date IS NULL     THEN 1 ELSE 0 END) AS pending,
       SUM(CASE WHEN ship_date IS NOT NULL THEN 1 ELSE 0 END) AS shipped
FROM orders;                       -- pending 1, shipped 2
```
Reading it: in the wrong query the order row for 10 appears three times (three items), so its `amount` is added three times. The right query keeps every join at the order grain. `COUNT(o.order_id)` returns 0 for Cy because the NULL-padded row has a NULL `order_id`.

### 5. Under the hood   *(optional at L4)*
- The optimizer treats `COUNT(DISTINCT x)` as a separate aggregate that needs either a sort (stream aggregate over sorted input) or a hash set per group. Mixed with ordinary aggregates it can force two aggregation passes; in SQL Server the plan often shows a *Stream Aggregate* below a *Hash Match (Aggregate)*.
- Pre-aggregating before the join usually reduces the join's input from N-lines to N-orders, so it is both *correct* and often *faster*. The optimizer can sometimes push a `GROUP BY` below a join itself, but only when it can prove the join key is unique; do not rely on it.
- A conditional aggregate scans the table once; N separate `COUNT(*) … WHERE` queries scan it N times. This is the book's "pivot-style aggregation without PIVOT".

### 6. Performance and design implications
- Aggregate at the lowest grain first, join afterwards — fewer rows flow into the join and the sort/hash.
- An index on the foreign key (`orders.cust_id`, `order_items.order_id`) lets the engine do merge/nested-loop joins and grouped aggregation without a sort.
- `COUNT(DISTINCT …)` over large inputs is memory-hungry; if approximate is fine use engine features (`APPROX_COUNT_DISTINCT` in SQL Server 2019+, `HyperLogLog` in others).
- Conditional aggregation over a wide, filtered, indexed set is typically one scan; check the actual plan for reads (`SET STATISTICS IO ON`).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `COUNT(o.order_id)` after `LEFT JOIN` | Counts only real matches | Parents with zero children must show 0 |
| `COUNT(DISTINCT o.order_id)` | De-duplicates fan-out | Quick fix for counts only |
| `SUM(CASE WHEN … THEN 1 ELSE 0 END)` | Several conditional counts in one scan | Status breakdowns, flags |
| `COUNT(CASE WHEN … THEN 1 END)` | Same idea; `CASE` returns NULL when false | Slightly shorter form |
| `COALESCE(x, 0)` | Turns NULL sums into 0 | Outer-joined aggregates |
| `APPROX_COUNT_DISTINCT` | Approximate distinct count | Huge tables, dashboards |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Pre-aggregate in a CTE/derived table | You only need one tiny count — a plain join is simpler |
| `COUNT(DISTINCT key)` | You also `SUM` a parent measure; the sum is still inflated |
| `LEFT JOIN` + `COUNT(right.col)` | You put a filter on the right table in `WHERE` — that turns it back into an inner join; put it in `ON` |
| Conditional aggregation | The conditions are hundreds of dynamic categories — use `PIVOT` / dynamic SQL |

### 9. Common mistakes
- `COUNT(*)` after a `LEFT JOIN` → customers with no orders show 1 → count a right-side column.
- Filtering the right table in `WHERE` (`WHERE o.status = 'OPEN'`) → zero-order customers vanish → move the predicate into the `ON` clause.
- Summing a parent amount after joining children → inflated revenue → pre-aggregate children or sum at the parent's grain.
- Using `COUNT(DISTINCT …)` as a magic fix everywhere → slow plans and still-wrong sums → fix the grain.
- Selecting non-grouped columns (e.g. `name`) next to the aggregate → error 8120 in SQL Server, or arbitrary values in lenient engines → add to `GROUP BY` or aggregate it.

### 10. 🎯 Interview kit
**30-second answer:** "A join multiplies rows, so counts after a one-to-many join count the multiplied rows. I aggregate the child table to the parent's grain first, then `LEFT JOIN` and count a right-side column so zero-child parents show 0. For several categories I use `SUM(CASE …)` in one scan."

**Structure a longer answer:** clarify the grain of the result → show how the join changes the grain → the fix (pre-aggregate / distinct / conditional) → trade-offs (distinct cost, outer-join `WHERE` trap) → a real scenario (revenue per customer with order lines).

**Likely follow-ups:**
- *Why does `COUNT(*)` return 1 for a customer with no orders?* → the outer join emits one NULL-padded row; `COUNT(*)` counts rows, `COUNT(col)` skips NULLs.
- *Count customers who placed at least one order?* → `COUNT(DISTINCT cust_id)` on orders, or `COUNT(*)` of customers with `EXISTS`.
- *Average orders per customer including zero?* → `COUNT(o.order_id) * 1.0 / COUNT(DISTINCT c.cust_id)` from the left-join, not `AVG` over customers with orders.
- *Percent of total per region?* → window aggregate `SUM(x) / SUM(SUM(x)) OVER ()` is better than a self-join to a grouped subquery.

**Signals interviewers look for:** you talk about *grain* unprompted, know the outer-join `WHERE` trap, and distinguish `COUNT(*)`, `COUNT(col)` and `COUNT(DISTINCT col)`.

### 11. Check yourself
- [ ] Explain why summing `orders.amount` after joining `order_items` inflates revenue.
- [ ] Write "orders and distinct products per customer, including customers with none".
- [ ] Rewrite two separate `COUNT … WHERE` queries as one conditional-aggregate query.
- [ ] Say what changes if you move `o.status = 'OPEN'` from `ON` to `WHERE` in a left join.
- [ ] Compute each region's share of total sales with a window function.
<!-- explainer:end -->

## Practice questions

- [Basic] [Write a query to count the total number of orders placed in a given year.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00747_Write_a_query_to_count_the_total_number_of_orders_placed_in_.md>)
- [Basic] [Write a query to find the total inventory value across all warehouses.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00887_Write_a_query_to_find_the_total_inventory_value_across_all_w.md>)
- [Intermediate] [Write a query to count how many employees each manager supervises.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00830_Write_a_query_to_count_how_many_employees_each_manager_super.md>)
- [Intermediate] [Write a query that returns each department with its employee count and average salary, sorted by count descending.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00852_Write_a_query_that_returns_each_department_with_its_employee.md>)
- [Intermediate] [Write a query to count the number of distinct products each customer has ever bought.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00877_Write_a_query_to_count_the_number_of_distinct_products_each_.md>)
- [Intermediate] [Write a query to count orders grouped by both product category and order status.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00892_Write_a_query_to_count_orders_grouped_by_both_product_catego.md>)
- [Intermediate] [Write a query to find the total and count of transactions per payment method in a single query.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00908_Write_a_query_to_find_the_total_and_count_of_transactions_pe.md>)
- [Deep] [Write a query using SUM with a CASE expression to count paid versus unpaid invoices.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00862_Write_a_query_using_SUM_with_a_CASE_expression_to_count_paid.md>)

## Related concepts

- [Aggregation Query Practice](<../SQL_Fundamentals/Aggregation_Query_Practice.md>) *(SQL Fundamentals)*
- [Basic Query Practice](<../../../TOOLS/MySQL/MySQL_Platform/Basic_Query_Practice.md>) *(MySQL Platform)*
- [Classic SQL Query Problems](<../../Database_Internals/DBMS_Concepts/Classic_SQL_Query_Problems.md>) *(DBMS Concepts)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Using CASE in GROUP BY/Aggregates
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Single-table queries
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Table expressions
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Grouping and Summarizing Data › The HAVING Clause
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Grouping and Summarizing Data › Solutions to Exercise 7-2: The GROUP BY Clause
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Discovering Windowing Functions › Summarizing Results with Window Aggregates

