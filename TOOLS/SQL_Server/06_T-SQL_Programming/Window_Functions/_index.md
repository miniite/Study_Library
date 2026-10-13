---
id: "TOOLS/SQL_Server/06_T-SQL_Programming/Window_Functions"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Window Functions

**L4 Advanced** · [↑ T-SQL Programming](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A window function computes a value for each row from a set of related rows (its *window*), defined by the `OVER` clause, without collapsing rows the way `GROUP BY` does. This lets you mix detail and aggregate in one query: ranks, running totals, previous/next values, percent of total.

### 1. The big picture
Before window functions, "show each order with its customer's total" needed a self-join or correlated subquery, and "running total" needed a cursor or a triangular self-join that read the table over and over. `GROUP BY` can't help: once you group, the detail rows are gone.

Window functions arrived in SQL Server 2005 (ranking functions and partitioned aggregates) and were completed in SQL Server 2012 (ordered aggregates with frames, `LAG`/`LEAD`, `FIRST_VALUE`/`LAST_VALUE`, distribution functions). SQL Server 2022 added the `WINDOW` clause for named windows. They are evaluated in the `SELECT` (and `ORDER BY`) phase of logical query processing. That one fact explains most of their rules. This lesson covers what they compute; the child lesson *Window Frames and Performance* covers frames in depth and how they run.

### 2. Core concepts and vocabulary
- **Window**: the set of rows a function looks at for the current row.
- **OVER clause**: defines the window. It has three optional parts: partitioning, ordering and framing.
- **PARTITION BY**: splits rows into independent groups. The function restarts for each partition. Without it, the whole result set is one partition.
- **Window ORDER BY**: orders rows *inside* each partition for the calculation. It has nothing to do with the order of the output; that still needs the query's `ORDER BY`.
- **Frame** (`ROWS` / `RANGE`): a subset of the partition relative to the current row, such as "all rows from the start of the partition to here".
- **Ranking functions**: `ROW_NUMBER`, `RANK`, `DENSE_RANK`, `NTILE`. Require window `ORDER BY`.
- **Window aggregate functions**: `SUM`, `COUNT`, `AVG`, `MIN`, `MAX` etc. with `OVER`.
- **Offset (analytic) functions**: `LAG`, `LEAD` (a row a given number of rows before/after), `FIRST_VALUE`, `LAST_VALUE` (first/last row in the frame).
- **Statistical (distribution) functions**: `PERCENT_RANK`, `CUME_DIST`, `PERCENTILE_CONT`, `PERCENTILE_DISC`.
- **Deterministic**: gives the same result every time. `ROW_NUMBER` with ties in the ordering is *nondeterministic*: tied rows can swap numbers between runs.

### 3. How it works, step by step
1. The query runs FROM, WHERE, GROUP BY and HAVING first. The window function sees the rows *after* those phases.
2. In the SELECT phase, for each window function, rows are partitioned by `PARTITION BY`.
3. Inside each partition, rows are ordered by the window `ORDER BY`.
4. For each row, the frame (if any) is determined, and the function is computed over it.
5. The result is attached as a new column. The number of rows does not change.

**Where they can appear:** only in `SELECT` and `ORDER BY`. You can't put them in `WHERE` or `GROUP BY`, because those phases run before windows are computed. To filter on a window result, compute it in a CTE or derived table and filter in the outer query.

**Ranking functions compared** (scores 100, 90, 90, 80):

| Score | ROW_NUMBER | RANK | DENSE_RANK | NTILE(2) |
|---|---|---|---|---|
| 100 | 1 | 1 | 1 | 1 |
| 90 | 2 | 2 | 2 | 1 |
| 90 | 3 | 2 | 2 | 2 |
| 80 | 4 | 4 | 3 | 2 |

- `ROW_NUMBER`: unique, consecutive. Ties broken arbitrarily unless you add a tiebreaker column.
- `RANK`: ties share a rank; leaves gaps after them.
- `DENSE_RANK`: ties share a rank; no gaps.
- `NTILE(n)`: splits rows into n groups as equal as possible; earlier groups get the extra rows.

**Window aggregates** without `ORDER BY` in `OVER` use the whole partition: `SUM(val) OVER (PARTITION BY custid)` is the customer total on every row. *With* `ORDER BY`, the default frame is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`: a running total. (Prefer writing `ROWS` explicitly; see the child lesson.)

### 4. Worked example
```sql
SELECT  o.custid, o.orderid, o.orderdate, o.val,
        -- ranking
        ROW_NUMBER() OVER (PARTITION BY o.custid ORDER BY o.orderdate, o.orderid) AS rownum,
        -- detail + aggregate together
        SUM(o.val) OVER (PARTITION BY o.custid)                         AS custtotal,
        CAST(100.0 * o.val / SUM(o.val) OVER (PARTITION BY o.custid)
             AS decimal(5,2))                                           AS pct_of_cust,
        -- running total
        SUM(o.val) OVER (PARTITION BY o.custid
                         ORDER BY o.orderdate, o.orderid
                         ROWS UNBOUNDED PRECEDING)                      AS runningtotal,
        -- previous and next order value
        LAG(o.val)  OVER (PARTITION BY o.custid ORDER BY o.orderdate, o.orderid) AS prevval,
        LEAD(o.val, 1, 0) OVER (PARTITION BY o.custid ORDER BY o.orderdate, o.orderid) AS nextval
FROM Sales.OrderValues AS o
ORDER BY o.custid, o.orderdate, o.orderid;
```
Read the output for one customer: `rownum` counts 1, 2, 3…; `custtotal` is the same on every row; `runningtotal` grows and equals `custtotal` on the last row; `prevval` is NULL on the first row (no previous row; you can give a default as `LAG(val, 1, 0)`), `nextval` is 0 on the last row because of the default.

**Filtering on a window result (top 2 orders per customer):**
```sql
WITH Ranked AS
(
    SELECT custid, orderid, val,
           ROW_NUMBER() OVER (PARTITION BY custid ORDER BY val DESC, orderid) AS rn
    FROM Sales.OrderValues
)
SELECT custid, orderid, val
FROM Ranked
WHERE rn <= 2;
```

**Removing duplicates, keeping the newest row:**
```sql
WITH D AS
(
    SELECT *, ROW_NUMBER() OVER (PARTITION BY email ORDER BY updated_at DESC) AS rn
    FROM dbo.Contacts
)
DELETE FROM D WHERE rn > 1;      -- deleting through the CTE deletes from dbo.Contacts
```

**Gaps and islands** (group consecutive dates): the difference between a date and its row number is constant within a run of consecutive days.
```sql
WITH G AS
(
    SELECT logdate,
           DATEADD(day, -ROW_NUMBER() OVER (ORDER BY logdate), logdate) AS grp
    FROM dbo.Logins
)
SELECT MIN(logdate) AS start_date, MAX(logdate) AS end_date, COUNT(*) AS days
FROM G
GROUP BY grp;
```

**Named windows (SQL 2022+):**
```sql
SELECT custid, orderid, val,
       SUM(val) OVER w AS runsum,
       AVG(val) OVER w AS runavg
FROM Sales.OrderValues
WINDOW w AS (PARTITION BY custid ORDER BY orderdate, orderid ROWS UNBOUNDED PRECEDING);
```

### 5. Under the hood
- **Plan shape.** A window function needs rows sorted by (partition columns, order columns). The plan shows a *Sort* (unless an index provides the order), then *Segment* (marks partition boundaries), then *Sequence Project* (for ranking functions) or *Window Spool* + *Stream Aggregate* (for framed aggregates). In batch mode (columnstore, or row-store batch mode in SQL 2019+ compat 150), all of this is replaced by a single *Window Aggregate* operator, which is much faster.
- **POC index.** An index on **P**artition columns, then **O**rder columns, **C**overing the rest (as INCLUDE) removes the Sort. Example: `CREATE INDEX ix ON Sales.Orders(custid, orderdate, orderid) INCLUDE (val);`
- **Several window functions with the same OVER** share one sort. Different `PARTITION BY` / `ORDER BY` combinations each need their own sort.
- **Determinism.** Without a unique tiebreaker, `ROW_NUMBER` may number tied rows differently on each run or under a parallel plan. Paging and deduplication then become unstable. Always add a unique column last (e.g. `orderid`).

### 6. Performance and design implications
- Window functions usually beat self-joins and correlated subqueries, which scale quadratically: a running total over n rows by self-join reads about n²/2 rows.
- The main cost is sorting. Provide a POC index or accept a sort (memory grant; watch for spills).
- Use `ROWS` explicitly for running aggregates. The default `RANGE` uses an on-disk worktable (details in the child lesson).
- `LAST_VALUE` with the default frame returns the *current* row, because the frame ends at the current row. Use `ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING`, or use `FIRST_VALUE` with descending order.

### 7. Commands and settings you should know
| Function / clause | What it does | When to use |
|---|---|---|
| `ROW_NUMBER()` | Unique sequence per partition | Paging, dedup, top-N per group |
| `RANK()` / `DENSE_RANK()` | Ranks with ties (gaps / no gaps) | Leaderboards |
| `NTILE(n)` | n roughly equal buckets | Quartiles, workload splitting |
| `SUM/COUNT/AVG … OVER` | Aggregates without grouping | Totals, percent of total, running totals |
| `LAG/LEAD(col, offset, default)` | Value from earlier/later row | Period-over-period comparisons |
| `FIRST_VALUE/LAST_VALUE` | First/last in frame | Opening/closing value; `IGNORE NULLS` in 2022+ |
| `PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY x) OVER (…)` | Interpolated median | Statistics |
| `WINDOW w AS (…)` (2022+) | Reusable window definition | Many functions, same window |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `ROW_NUMBER` + CTE for top-N per group | Small N with a good index: `CROSS APPLY (SELECT TOP n …)` may be cheaper |
| Window aggregate for detail + total | Only totals needed → plain `GROUP BY` |
| `LAG/LEAD` instead of self-joins on "previous row" | — |
| `RANK` for fair ties | Need exactly n rows → `ROW_NUMBER` |
| Window functions in reports | Filtering needs a CTE; can't use in `WHERE` |

### 9. Common mistakes
- Putting a window function in `WHERE` → error 4108 → compute in a CTE, filter outside.
- Expecting the window `ORDER BY` to sort the output → unordered result → add the query `ORDER BY`.
- `ROW_NUMBER` without a unique tiebreaker → unstable results → add the key column.
- Using `LAST_VALUE` with the default frame → returns the current row → specify the frame.
- Running total with the default `RANGE` → slow worktable in tempdb, and wrong-looking results when order values tie → `ROWS UNBOUNDED PRECEDING`.
- Using `COUNT(DISTINCT x) OVER (…)` → not supported → `DENSE_RANK` trick (`DENSE_RANK() OVER (… ORDER BY x ASC) + DENSE_RANK() OVER (… ORDER BY x DESC) - 1`).

### 10. 🎯 Interview kit
**30-second answer:** Window functions compute a value per row over a related set of rows defined by OVER: PARTITION BY groups, ORDER BY orders inside the group, and a ROWS/RANGE frame narrows it. Unlike GROUP BY they keep every detail row. Families: ranking (ROW_NUMBER, RANK, DENSE_RANK, NTILE), aggregates (running totals, percent of total), offset (LAG, LEAD, FIRST_VALUE) and distribution functions. They're evaluated in the SELECT phase, so you filter them via a CTE.

**Structure a longer answer:** what problem they solve (detail + aggregate) → OVER parts → function families → a top-N or gaps-and-islands example → performance (POC index, ROWS vs. RANGE, batch mode).

**Likely follow-ups:**
- *RANK vs. DENSE_RANK?* → gaps after ties vs. no gaps.
- *Why can't I use ROW_NUMBER in WHERE?* → logical order: WHERE runs before SELECT.
- *How to speed up?* → POC index removes the sort; ROWS frame; batch mode.
- *Delete duplicates?* → ROW_NUMBER in a CTE, delete rn > 1.

**Signals interviewers look for:** you mention determinism/tiebreakers; you know the default frame; you can solve top-N per group and gaps and islands.

### 11. Check yourself
- [ ] Fill in ROW_NUMBER, RANK, DENSE_RANK and NTILE(3) for a list with ties.
- [ ] Write a query showing each order, its customer total and its percent of that total.
- [ ] Return the latest order per customer two ways: ROW_NUMBER and CROSS APPLY TOP 1.
- [ ] Solve a gaps-and-islands problem on dates.
- [ ] Explain why LAST_VALUE "doesn't work" with the default frame.
<!-- explainer:end -->

## In this section (study in order)

1. [Window Frames and Performance](<Window_Frames_and_Performance/_index.md>) (L5 Expert)

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Window Function
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Categories of Window Functions
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Aggregate Window Functions
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Using Window Aggregates
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Window Functions
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Discovering Window Functions
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Summary › � R
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Thinking About the Window
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — T-SQL for data analysis
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Putting It All Together

