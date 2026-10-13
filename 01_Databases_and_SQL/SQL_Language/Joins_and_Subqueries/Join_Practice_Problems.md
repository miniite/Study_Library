---
id: "01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Join_Practice_Problems"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries"
tools: []
---

# Join Practice Problems

**L4 Advanced** · [↑ Joins and Subqueries](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A workbook of the join patterns that recur in practice — inner vs outer to keep zero-match rows, anti-joins for "never", counting without fan-out, self-joins for hierarchies, and sorting joined output — each with the reasoning that picks the right form.

### 1. The big picture
Join problems are mostly pattern recognition. Once you can name the pattern ("keep all parents", "find the missing", "count children without inflating", "compare a row to its neighbour in the same table") the SQL follows. This lesson walks through the exercise set that textbooks use (customers/orders/order details, products/sales order lines, employees/managers) and extracts the thinking.

It builds on *Joins vs Subqueries and Join Types* (the definitions) and *Counting with Joins* (fan-out). Here the focus is working the problems end to end: restate the requirement, decide the grain, choose the join type, write, then check the row count against a known figure.

### 2. Core concepts and vocabulary
- **Preserved side** — the table whose rows must all appear in an outer join result.
- **Join predicate vs filter predicate** — `ON` decides matching; `WHERE` decides which result rows survive.
- **Fan-out** — row multiplication by a one-to-many join.
- **Anti-join** — parents with no matching child.
- **Self-join** — joining a table to itself under two aliases (hierarchies, "previous row" comparisons).
- **Cross join** — all combinations; useful for scaffolding (every customer × every month).
- **Row-count check** — sanity test: inner join result count vs outer join count differ by the number of unmatched rows (e.g. book exercise: 830 inner rows vs 832 outer rows, because two customers never ordered).

### 3. How it works, step by step
Method for every exercise:
1. **State the grain**: one row per what?
2. **Name the preserved side**: do rows with no match need to show up?
3. **Pick the join**: `INNER` for matched only, `LEFT` to preserve, `FULL` rarely, `CROSS` for combinations, `EXISTS` for tests.
4. **Place filters**: preserved-side filters in `WHERE`; optional-side filters in `ON`.
5. **Aggregate at the right grain**: `COUNT(DISTINCT parent_key)` or pre-aggregation.
6. **Verify**: compare counts with a simple independent query.

### 4. Worked example
Problems and solutions (schema as in the standard sales sample: `Customers(custid, companyname, country)`, `Orders(orderid, custid, orderdate)`, `OrderDetails(orderid, productid, qty)`).

```sql
-- P1. Customers (USA) with number of orders and total quantity.
--    Trap: joining details inflates the order count -> COUNT(DISTINCT).
SELECT C.custid,
       COUNT(DISTINCT O.orderid) AS numorders,
       SUM(OD.qty)               AS totalqty
FROM Customers C
JOIN Orders O        ON O.custid  = C.custid
JOIN OrderDetails OD ON OD.orderid = O.orderid
WHERE C.country = N'USA'
GROUP BY C.custid;
-- qty belongs to the order line, so summing it is safe; the order id repeats per line.

-- P2. All customers, with their orders if any.
SELECT C.custid, C.companyname, O.orderid, O.orderdate
FROM Customers C LEFT JOIN Orders O ON O.custid = C.custid;
-- inner join gives 830 rows; the left join gives 832 (two customers never ordered).

-- P3. Only the customers WITHOUT orders.
SELECT C.custid, C.companyname
FROM Customers C LEFT JOIN Orders O ON O.custid = C.custid
WHERE O.orderid IS NULL;       -- test the right side's key

-- P4. Products never ordered (book exercise: product left join order detail, keep NULLs).
SELECT P.ProductID, P.Name
FROM Product P LEFT JOIN OrderDetail D ON D.ProductID = P.ProductID
WHERE D.SalesOrderID IS NULL;

-- P5. Employees and their managers (self-join; preserve the CEO).
SELECT e.EmployeeName AS Employee, m.EmployeeName AS Manager
FROM Employees e LEFT JOIN Employees m ON e.ManagerID = m.EmployeeID;

-- P6. Orders sorted by customer then newest first; ORDER BY comes last and may use any column.
SELECT C.companyname, O.orderid, O.orderdate
FROM Customers C JOIN Orders O ON O.custid = C.custid
ORDER BY C.companyname, O.orderdate DESC;

-- P7. Customer revenue including zeros.
SELECT C.custid, COALESCE(SUM(OD.qty * OD.unitprice), 0) AS revenue
FROM Customers C
LEFT JOIN Orders O        ON O.custid = C.custid
LEFT JOIN OrderDetails OD ON OD.orderid = O.orderid
GROUP BY C.custid;
```
Check P7: `COALESCE` converts the NULL sum of a customer with no rows; a `SUM` over a joined row that is entirely NULL yields NULL, not 0.

### 5. Under the hood   *(optional at L4)*
- `LEFT JOIN … WHERE r.pk IS NULL` is recognised by the optimizer as an **anti-semi-join**, the same plan shape as `NOT EXISTS`.
- A chain of left joins is evaluated in the written order logically; reordering outer joins is restricted (they are not freely commutative), so the written structure can limit the plan.
- `ORDER BY` after a join may be satisfied by an index that matches the *join output order* (merge join) so no explicit sort is needed.

### 6. Performance and design implications
- Index every foreign key used in joins; unindexed FKs force scans and hash joins.
- When only existence is needed use `EXISTS` instead of a full join.
- Pre-aggregating a child table before joining reduces rows flowing into the join and the aggregate.
- Use the plan and `SET STATISTICS IO` to compare alternative forms; don't guess.
- Always `ORDER BY` for determinism — join output order is unspecified.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `INNER JOIN` | Matches only | Both sides required |
| `LEFT JOIN … IS NULL` | Missing-row finder | "Never ordered" |
| `COUNT(DISTINCT …)` | De-duplicates fan-out | Counts after a 3-table join |
| `COALESCE(x, 0)` | Zero-fill outer-joined aggregates | Reports |
| `CROSS JOIN` | All combinations | Calendar scaffolds |
| `ORDER BY` | Final deterministic sort | Always for presentation |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `LEFT JOIN` + `IS NULL` | The right table's key can be NULL by design — test a NOT NULL column |
| `NOT EXISTS` | You need columns from the right (impossible by definition) |
| `COUNT(DISTINCT)` repair | You also sum a parent-level measure |
| Self-join for a single level of hierarchy | Unknown depth — use a recursive CTE |

### 9. Common mistakes
- Inner join where zero-match rows were required → silent data loss → outer join, verify counts.
- Filter on the optional table in `WHERE` → outer join becomes inner → move it to `ON`.
- Counting `COUNT(*)` across a 3-table join → inflated → `COUNT(DISTINCT parent_id)`.
- Missing a join predicate → Cartesian product (rows = A × B) → always check row counts.
- Ambiguous column names → error → qualify with aliases.

### 10. 🎯 Interview kit
**30-second answer:** "First I decide the grain and which side must be preserved. Left join keeps parents with no children; `IS NULL` on a right-side key finds the missing ones. After a one-to-many join I count distinct parent keys or pre-aggregate, and I always sanity-check row counts."

**Structure a longer answer:** grain → preserved side → join type → filter placement → aggregation → verification and indexes.

**Likely follow-ups:**
- *Where does `ORDER BY` go?* → last, after all joins; may reference aliases and non-selected columns (unless `DISTINCT`).
- *Customers with zero orders listed with count 0?* → `LEFT JOIN` + `COUNT(o.order_id)`.
- *Total revenue per customer?* → join, `GROUP BY` customer, `SUM` the line amount.
- *Self-join use cases?* → manager hierarchy, pairs in same table, previous/next comparisons.

**Signals interviewers look for:** you check counts, you notice fan-out, and you know the `ON` vs `WHERE` difference.

### 11. Check yourself
- [ ] Solve P1–P7 again from the schema without looking, then compare.
- [ ] Explain why `COUNT(DISTINCT orderid)` is needed in P1 but not for `SUM(qty)`.
- [ ] Produce a customer × month grid showing zeros using a cross join and left join.
- [ ] Find employees earning more than their manager with a self-join.
<!-- explainer:end -->

## Practice questions

- [Basic] [Can you order or sort results that come from a JOIN? Where does ORDER BY go?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30428_Can_you_order_or_sort_results_that_come_from_a_JOIN_Where_do.md>)
- [Intermediate] [How do you find customers who have never placed an order using a JOIN?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30336_How_do_you_find_customers_who_have_never_placed_an_order_usi.md>)
- [Intermediate] [How do you count matching rows per group after a join, for example orders per customer?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30412_How_do_you_count_matching_rows_per_group_after_a_join_for_ex.md>)
- [Intermediate] [How do you combine a JOIN with GROUP BY to compute total revenue per customer?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30450_How_do_you_combine_a_JOIN_with_GROUP_BY_to_compute_total_rev.md>)
- [Intermediate] [You need each customer's order count including zero — why does a LEFT JOIN + COUNT(*) overcount and how does COUNT(child.id) fix it?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30301_You_need_each_customer_s_order_count_including_zero_why_does.md>)
- [Deep] [When joining three tables Orders Customers and Products what order does the join get evaluated in logically?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30369_When_joining_three_tables_Orders_Customers_and_Products_what.md>)
- [Deep] [How do you write a query to get customers along with their most recent order date?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30418_How_do_you_write_a_query_to_get_customers_along_with_their_m.md>)
- [Deep] [How do you write a query to fetch the latest record per group, such as the latest login per user?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30478_How_do_you_write_a_query_to_fetch_the_latest_record_per_grou.md>)

## Related concepts

- [Basic Query Practice](<../../../TOOLS/MySQL/MySQL_Platform/Basic_Query_Practice.md>) *(MySQL Platform)*
- [Aggregation Query Practice](<../SQL_Fundamentals/Aggregation_Query_Practice.md>) *(SQL Fundamentals)*
- [Counting with Joins](<../Aggregation/Counting_with_Joins.md>) *(Aggregation)*

## References

- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Joins
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Subqueries
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Joining Tables › Adding a Table to the Right Side of a LEFT JOIN
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Joining Tables › Self-Joins
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Joining Tables › Solutions to Exercise 5-2: Using OUTER JOIN
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Building on Subqueries, Common Table Expressions, and Unions › Using a Common Table Expression to Solve a Complicated Join Problem
- *Grokking Relational Database Design* — Qiang Hao, Michail Tsikerdekis — Grokking Relational Database Design (cont.)

