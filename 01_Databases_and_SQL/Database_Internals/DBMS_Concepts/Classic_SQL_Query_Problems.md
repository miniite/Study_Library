---
id: "01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Classic_SQL_Query_Problems"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Internals/DBMS_Concepts"
tools: ["MySQL", "SQL_Server"]
---

# Classic SQL Query Problems

**L4 Advanced** · [↑ DBMS Concepts](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A small family of query *shapes* — top-N per group, running totals, gaps and islands, de-duplication, pivot/unpivot, grouping sets, relational division, "not exists" — recur everywhere; learn the pattern behind each and the problem becomes mechanical.

### 1. The big picture
Interview and real-world SQL problems look different on the surface but reduce to a handful of patterns. Learning the *patterns* (and the window-function/set-based tools they rely on) is far more productive than memorizing solutions. This lesson walks through the classics with a reasoning template: **identify the grain → identify the group/partition → pick the operator (window, join, aggregate, anti-join) → verify with edge cases (ties, NULLs, empty groups)**.

Also keep the clarity rule in mind: correctness and readability first (explicit joins, meaningful aliases, indentation), optimize only after measuring.

### 2. Core concepts and vocabulary
- **Grain** — what one row represents; most wrong answers come from joining at a different grain than assumed.
- **Window function** — computes over a set of rows related to the current row without collapsing them (`ROW_NUMBER`, `RANK`, `DENSE_RANK`, `SUM() OVER`, `LAG/LEAD`).
- **`PARTITION BY` / `ORDER BY` / frame** — define the window groups, order, and range (`ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`).
- **Anti-join** — rows in A with no match in B (`NOT EXISTS`, `LEFT JOIN … IS NULL`).
- **Semi-join** — rows in A that have at least one match (`EXISTS`).
- **Self-join** — a table joined to itself (hierarchies, pairs, previous row).
- **Gaps and islands** — finding missing values (gaps) or consecutive runs (islands) in a sequence.
- **Grouping sets / `ROLLUP` / `CUBE`** — multiple aggregation levels in one query; `GROUPING_ID` tells which level a row is.
- **Relational division** — "entities related to *all* items of a set".
- **Pivot / unpivot** — rows ↔ columns.
- **Tie handling** — `ROW_NUMBER` breaks ties arbitrarily; `RANK`/`DENSE_RANK`/`TOP WITH TIES` keep them.

### 3. How it works, step by step
**Template:** (1) state the output grain; (2) list the inputs and their keys; (3) decide whether you need per-row context (window) or collapsed groups (aggregate); (4) write the query in layers with CTEs; (5) test with ties, NULLs, empty groups, duplicates.

**The classics and their pattern:**

| Problem | Pattern |
|---|---|
| Nth highest salary | `DENSE_RANK() OVER (ORDER BY salary DESC)` and filter, or `OFFSET n-1 FETCH 1` on `DISTINCT` |
| Top N per group | `ROW_NUMBER() OVER (PARTITION BY g ORDER BY x DESC)` ≤ N (or `CROSS APPLY TOP`) |
| Duplicates | `GROUP BY cols HAVING COUNT(*) > 1`; delete via `ROW_NUMBER()` > 1 in a CTE |
| Running total | `SUM(x) OVER (PARTITION BY g ORDER BY d ROWS UNBOUNDED PRECEDING)` |
| Previous/next row, change detection | `LAG/LEAD` |
| Gaps and islands | `value - ROW_NUMBER()` constant within an island |
| Customers with no orders | `NOT EXISTS` / `LEFT JOIN … IS NULL` |
| Customers who bought *all* products | `GROUP BY c HAVING COUNT(DISTINCT p) = (SELECT COUNT(*) FROM Products)` |
| Employees earning more than their manager | self-join on manager id |
| Median / percentiles | `PERCENTILE_CONT … WITHIN GROUP` or row-number from both ends |
| Pivot | conditional aggregation `SUM(CASE WHEN … THEN … END)` or `PIVOT` |
| Unpivot | `CROSS APPLY (VALUES …)` or `UNPIVOT` |
| Totals at several levels | `GROUPING SETS`, with `GROUPING_ID` as label |

### 4. Worked example
```sql
-- Top 3 orders per customer (ties broken by OrderID)
WITH r AS (
  SELECT CustomerID, OrderID, OrderTotal,
         ROW_NUMBER() OVER (PARTITION BY CustomerID ORDER BY OrderTotal DESC, OrderID) AS rn
  FROM Sales.Orders)
SELECT CustomerID, OrderID, OrderTotal FROM r WHERE rn <= 3;

-- Remove duplicates, keep the earliest row
WITH d AS (
  SELECT *, ROW_NUMBER() OVER (PARTITION BY Email ORDER BY CreatedAt, CustomerID) AS rn
  FROM dbo.Customers)
DELETE FROM d WHERE rn > 1;

-- Running total and month-over-month change
SELECT MonthStart, Revenue,
       SUM(Revenue) OVER (ORDER BY MonthStart ROWS UNBOUNDED PRECEDING) AS RunningRevenue,
       Revenue - LAG(Revenue) OVER (ORDER BY MonthStart)                AS MoM
FROM dbo.MonthlyRevenue;

-- Islands: consecutive login days per user
WITH x AS (
  SELECT UserID, LoginDate,
         DATEADD(DAY, -ROW_NUMBER() OVER (PARTITION BY UserID ORDER BY LoginDate), LoginDate) AS grp
  FROM (SELECT DISTINCT UserID, CAST(LoginAt AS date) AS LoginDate FROM dbo.Logins) l)
SELECT UserID, MIN(LoginDate) AS StartDate, MAX(LoginDate) AS EndDate, COUNT(*) AS Days
FROM x GROUP BY UserID, grp;

-- Customers with no orders in 2026 (NULL-safe anti-join)
SELECT c.CustomerID
FROM Sales.Customers c
WHERE NOT EXISTS (SELECT 1 FROM Sales.Orders o
                  WHERE o.CustomerID = c.CustomerID AND o.OrderDate >= '2026-01-01');

-- Multiple aggregation levels in one pass
SELECT empid, custid, YEAR(orderdate) AS orderyear, SUM(qty) AS sumqty,
       GROUPING_ID(empid, custid, YEAR(orderdate)) AS groupingset
FROM dbo.Orders
GROUP BY GROUPING SETS ((empid, custid, YEAR(orderdate)),
                        (empid, YEAR(orderdate)),
                        (custid, YEAR(orderdate)));
```
**How to read:** islands work because consecutive dates minus a consecutive row number give the same constant; each gap changes the constant, starting a new island. The anti-join uses `NOT EXISTS` because `NOT IN` returns nothing if the subquery produces a `NULL`.

### 5. Under the hood
- **Window functions** are computed by a *Window Spool/Segment/Sequence Project* sequence in the plan after a sort (or an index providing order). A supporting index on `(PartitionCol, OrderCol)` with included columns avoids the sort. `ROWS` frames are much faster than the default `RANGE` frame with `ORDER BY` (which uses a disk-spilled on-disk spool in SQL Server) — always write `ROWS`.
- **`NOT IN` vs `NOT EXISTS` vs `LEFT JOIN`.** `NOT EXISTS` and `LEFT JOIN … IS NULL` give the same plan (anti-semi-join); `NOT IN` has three-valued-logic traps with `NULL`.
- **Correlated subqueries** are often rewritten by the optimizer into joins, but not always; test on large data.
- **`DISTINCT` vs `GROUP BY`** are equivalent to the optimizer for plain deduplication.
- **Join fan-out.** Joining a header to two child tables multiplies rows (orders × lines × payments); aggregate each child separately first, then join.
- **Pivot with unknown columns** requires dynamic SQL (see the Dynamic SQL lesson).

### 6. Performance and design implications
- Provide indexes matching `PARTITION BY` + `ORDER BY` for window queries; use covering/included columns.
- Prefer set-based over cursor/loop solutions; running totals with window frames are linear, self-join running totals are quadratic.
- Avoid functions on indexed columns in predicates (`WHERE YEAR(d) = 2026` → `d >= '2026-01-01' AND d < '2027-01-01'`).
- Use `TOP` + `ORDER BY` with a supporting index for top-N.
- Check plans for spills (Sort/Hash warnings) on big windows.
- Write for clarity: explicit `JOIN … ON`, meaningful aliases, one clause per line.

### 7. Commands and settings you should know
| Command / feature | What it does | When to use |
|---|---|---|
| `ROW_NUMBER/RANK/DENSE_RANK/NTILE` | Ranking | Top-N, dedupe, buckets |
| `LAG/LEAD/FIRST_VALUE/LAST_VALUE` | Neighboring rows | Change detection, deltas |
| `SUM() OVER (… ROWS …)` | Running aggregates | Cumulative totals |
| `GROUPING SETS/ROLLUP/CUBE` | Multi-level aggregation | Subtotals, reports |
| `CROSS/OUTER APPLY` | Per-row table expression | Top-N per group, unpivot |
| `EXISTS/NOT EXISTS` | Semi/anti join | Presence/absence tests |
| `PIVOT/UNPIVOT`, conditional aggregation | Reshaping | Cross-tabs |
| `STRING_AGG`, `PERCENTILE_CONT` | Aggregates | Lists, medians |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `ROW_NUMBER` for deterministic top-1 | Ties should all be returned (`RANK`/`WITH TIES`) |
| `NOT EXISTS` | `NOT IN` against nullable columns |
| Window function | You only need a collapsed total (a plain `GROUP BY` is cheaper) |
| `APPLY` top-N | Many groups and no supporting index — a window with sort may win |
| Conditional aggregation pivot | Column list is dynamic |

### 9. Common mistakes
- Joining at the wrong grain → inflated sums → aggregate children before joining.
- `NOT IN` with NULLs → empty result → `NOT EXISTS`.
- `ROW_NUMBER` without a tiebreaker → nondeterministic results → add unique column to `ORDER BY`.
- Default window frame with `ORDER BY` → slow and surprising ties → specify `ROWS`.
- Filtering windows in `WHERE` → not allowed → wrap in CTE and filter outside.
- Counting `COUNT(col)` expecting rows → NULLs skipped → `COUNT(*)` or be explicit.
- Forgetting empty groups → missing rows → left join from a calendar/dimension.

### 10. 🎯 Interview kit
**30-second answer:** Most classic problems are pattern matches: top-N per group is `ROW_NUMBER` over a partition, running totals are a windowed `SUM`, absence is `NOT EXISTS`, "all of" is relational division with `HAVING COUNT`, and islands come from value minus row number. I state the grain, pick the pattern, then test ties and NULLs.

**Structure a longer answer:** clarify (grain, ties, NULLs, size) → pattern → write in CTE layers → complexity and indexes → edge cases.

**Likely follow-ups:**
- *Second-highest salary without window functions?* → `MAX(salary) WHERE salary < (SELECT MAX…)` or `OFFSET/FETCH`.
- *Delete duplicates but keep one?* → CTE with `ROW_NUMBER`, delete `rn > 1`.
- *Why `ROWS` rather than `RANGE`?* → semantics with ties and performance.
- *Pivot unknown categories?* → dynamic SQL with `QUOTENAME`.
- *How to speed a window query?* → index on partition/order columns, avoid spills.

**Signals interviewers look for:** correctness on ties and NULLs, clear structure, awareness of performance.

### 11. Check yourself
- [ ] Solve top-3 per group with and without `ROW_NUMBER`.
- [ ] Find consecutive-day streaks per user.
- [ ] Explain why `NOT IN` fails with NULLs.
- [ ] Write a customers-who-bought-all-products query.
- [ ] Produce subtotals and grand total with `GROUPING SETS` and label the levels.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../../../TOOLS/MySQL/_index.md>)
- [SQL Server](<../../../TOOLS/SQL_Server/_index.md>)

## Practice questions

- [Basic] [How would you write a SQL query to detect duplicate business keys that should be unique in a dimension table?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22576_How_would_you_write_a_SQL_query_to_detect_duplicate_business.md>)
- [Intermediate] [How would you deduplicate records that arrived multiple times using ROW_NUMBER partitioned by the business key?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22581_How_would_you_deduplicate_records_that_arrived_multiple_time.md>)
- [Intermediate] [How would you design a query to compute daily new versus returning users from an events table?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22632_How_would_you_design_a_query_to_compute_daily_new_versus_ret.md>)
- [Intermediate] [Write a query to find duplicate rows (or duplicate emails) in a table.](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22171_Write_a_query_to_find_duplicate_rows_or_duplicate_emails_in_.md>)
- [Intermediate] [Write a query using LAG or LEAD to compute the difference between consecutive daily sales.](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22269_Write_a_query_using_LAG_or_LEAD_to_compute_the_difference_be.md>)
- [Intermediate] [How do you calculate the number of days between two dates in SQL?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22376_How_do_you_calculate_the_number_of_days_between_two_dates_in.md>)
- [Deep] [How would you compute retention: the number of users active in month N who were also active in month N minus 1?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22593_How_would_you_compute_retention_the_number_of_users_active_i.md>)
- [Deep] [Write a query to compute a running total of sales over time.](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22202_Write_a_query_to_compute_a_running_total_of_sales_over_time.md>)

## Related concepts

- [Basic Query Practice](<../../../TOOLS/MySQL/MySQL_Platform/Basic_Query_Practice.md>) *(MySQL Platform)*
- [Scalar Subqueries](<../../SQL_Language/SQL_Fundamentals/Scalar_Subqueries.md>) *(SQL Fundamentals)*
- [Aggregation Query Practice](<../../SQL_Language/SQL_Fundamentals/Aggregation_Query_Practice.md>) *(SQL Fundamentals)*
- [Advanced Aggregation](<../../SQL_Language/Aggregation/Advanced_Aggregation.md>) *(Aggregation)*
- [Gaps and Islands](<../../SQL_Language/SQL_Fundamentals/Gaps_and_Islands.md>) *(SQL Fundamentals)*

## References

- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — T-SQL for data analysis
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Data ��difi�ati�n
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — SQL Graph
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Multi-Column Distinct
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 16: Best Practices for T-SQL Development and Database Design › Commenting Standards
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 16: Best Practices for T-SQL Development and Database Design › Write for Clarity First, Optimize Second
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Segment and rowgroup elimination
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — SQL Analytics Using Amazon Redshift › Semi-Structured Data Analysis
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.)
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Discovering Windowing Functions › Understanding the Difference Between ROWS and RANGE
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Discovering Windowing Functions › Solutions to Exercise 8-3: Understanding the Difference Between ROWS and RANGE
- *SQL Server Tutorial* (YouTube series) — Part 102: Rollup in SQL Server ([watch](https://www.youtube.com/watch?v=HLTdfCtfIJs))

