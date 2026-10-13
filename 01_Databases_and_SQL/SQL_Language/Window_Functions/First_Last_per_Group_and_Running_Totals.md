---
id: "01_Databases_and_SQL/SQL_Language/Window_Functions/First_Last_per_Group_and_Running_Totals"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/Window_Functions"
tools: []
---

# First Last per Group and Running Totals

**L4 Advanced** · [↑ Window Functions](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** FIRST_VALUE/LAST_VALUE fetch a column from the edge of a frame, and aggregates with an ordered, framed window produce running and reverse-running totals — both are controlled by the frame, and the default frame is the trap.

### 1. The big picture
Two everyday needs: "show each row next to the first (or last) row of its group" and "show a cumulative total up to this row". Before SQL Server 2012 both needed self-joins or cursors. Frames fix that: `ROWS BETWEEN <start> AND <end>` tells the function exactly which rows around the current one to include.

Mastering the frame removes the three famous bugs: LAST_VALUE returning the current row, running totals that repeat on ties, and slow RANGE spools.

### 2. Core concepts and vocabulary
- **Frame** — the subset of the ordered partition used for the current row.
- **ROWS** — frame measured in physical row offsets.
- **RANGE** — frame measured in values; rows with equal ORDER BY value (peers) enter together.
- **UNBOUNDED PRECEDING / FOLLOWING** — partition start / end.
- **CURRENT ROW**, **n PRECEDING**, **n FOLLOWING** — offsets from the current row.
- **Running total** — `SUM` from partition start to current row.
- **Reverse running total** — from current row to partition end (remaining balance).
- **FIRST_VALUE / LAST_VALUE** — column value from the first / last row of the frame.
- **WINDOW clause** — names a shared window specification.

### 3. How it works, step by step
1. Partition and order the rows.
2. For each current row, compute the frame boundaries.
3. Apply the aggregate / value function over the frame rows only.

Defaults — the part everyone forgets:
- No ORDER BY: frame = whole partition.
- ORDER BY present, no frame: `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`.

So `FIRST_VALUE(x) OVER (PARTITION BY g ORDER BY d)` works (frame starts at partition start). `LAST_VALUE(x) OVER (PARTITION BY g ORDER BY d)` returns the **current row's** value (the frame ends at the current row/peers). To get the true last row you must extend the frame to `UNBOUNDED FOLLOWING`.

### 4. Worked example
```sql
SELECT custid, orderid, orderdate, val,
  -- running and reverse running totals
  SUM(val) OVER (PARTITION BY custid ORDER BY orderdate, orderid
                 ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total,
  SUM(val) OVER (PARTITION BY custid ORDER BY orderdate, orderid
                 ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING) AS reverse_total,
  -- first and last order value per customer
  FIRST_VALUE(val) OVER (PARTITION BY custid ORDER BY orderdate, orderid
                 ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS first_val,
  LAST_VALUE(val)  OVER (PARTITION BY custid ORDER BY orderdate, orderid
                 ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS last_val,
  -- running count and ratio
  COUNT(*) OVER (PARTITION BY custid ORDER BY orderdate, orderid
                 ROWS UNBOUNDED PRECEDING) AS order_no,
  val * 1.0 / SUM(val) OVER (PARTITION BY custid) AS share_of_customer
FROM Sales.OrderValues;
```
With the same window repeated, name it:
```sql
SELECT custid, orderid, val,
       SUM(val) OVER W AS runsum, AVG(val) OVER W AS runavg
FROM Sales.OrderValues
WINDOW W AS (PARTITION BY custid ORDER BY orderdate, orderid
             ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW);
```
Remaining balance in one pass: `total − running` = `SUM(val) OVER (PARTITION BY custid) − SUM(val) OVER (... ROWS UNBOUNDED PRECEDING)`, or simply the reverse total minus current row.

First crossing of a threshold:
```sql
WITH r AS (SELECT custid, orderdate, val,
                  SUM(val) OVER (PARTITION BY custid ORDER BY orderdate, orderid
                                 ROWS UNBOUNDED PRECEDING) AS run
           FROM Sales.OrderValues),
f AS (SELECT *, ROW_NUMBER() OVER (PARTITION BY custid ORDER BY orderdate) AS n
      FROM r WHERE run > 10000)
SELECT * FROM f WHERE n = 1;
```

### 5. Under the hood
- `ROWS` frames use an in-memory accumulator that adds the entering row and (for sliding frames) subtracts the leaving row; for `UNBOUNDED PRECEDING … CURRENT ROW` it is one pass.
- `RANGE` frames in SQL Server, when a Window Spool is needed, spool rows to a worktable in tempdb: expect many logical reads under `Worktable`. `ROWS` avoids it ("fast-track" in-memory spool).
- MIN/MAX over a sliding frame can't be maintained by subtraction, so the engine may re-scan the frame; cost grows with frame size.
- FIRST_VALUE with default frame is cheap; LAST_VALUE needs the frame to reach the end.
- Frames only apply to aggregates and FIRST_VALUE/LAST_VALUE; ROW_NUMBER, RANK, LAG, LEAD ignore any frame.

### 6. Performance and design implications
- Always write `ROWS` explicitly for running totals; with a unique ORDER BY it is both correct and fast.
- Index `(partition, order) INCLUDE (measure)` avoids the sort; watch for `Worktable` in `SET STATISTICS IO`.
- Restricting the frame (e.g. last 7 rows) bounds work per row; unbounded sliding MIN/MAX on huge frames is expensive.
- For very large running totals consider pre-aggregating to day/month first, then windowing over the smaller set.
- Running totals recomputed from scratch on each query are fine; materialising them goes stale when history changes (late CDC updates) — design reload windows.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SUM(x) OVER (... ROWS UNBOUNDED PRECEDING)` | Running total | Cumulative sales, balances |
| `... ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING` | Reverse running total | Remaining quantity |
| `FIRST_VALUE(x) OVER (...)` | Value from first row | Opening balance, first purchase |
| `LAST_VALUE(x) OVER (... UNBOUNDED FOLLOWING)` | Value from last row | Latest status per group |
| `IGNORE NULLS` | Skip NULLs (Oracle, Snowflake, BigQuery, Spark; SQL Server 2022) | Forward-fill |
| `WINDOW w AS (...)` | Named window | Several functions, one spec |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| FIRST/LAST_VALUE for a column from the edge row | Two columns from the edge → ROW_NUMBER = 1 filter gives the whole row |
| ROWS frame | You deliberately want peers aggregated together (RANGE) |
| One window for many measures | Different orderings needed → separate sorts |

### 9. Common mistakes
- LAST_VALUE with the default frame → returns current row → add `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` (or `CURRENT ROW AND UNBOUNDED FOLLOWING`).
- Running total over non-unique ORDER BY with default RANGE → tied rows show the same total → unique ORDER BY + ROWS.
- Missing PARTITION BY → one running total across all customers.
- Integer division in a ratio (`val / SUM(val) OVER ()`) → 0 → multiply by 1.0 or CAST.
- Using MAX(x) when you need the *row* with the max → loses the other columns → ROW_NUMBER or FIRST_VALUE.
- Running total across a date gap treated as continuous → fill the calendar first if period matters.

### 10. 🎯 Interview kit
**30-second answer:** A running total is `SUM(x) OVER (PARTITION BY g ORDER BY t ROWS UNBOUNDED PRECEDING)`. The frame defines which rows participate; FIRST_VALUE/LAST_VALUE use the frame ends, so LAST_VALUE needs an explicit frame to the partition's end.

**Structure a longer answer:** clarify partition/order → write the frame → explain default RANGE and peers → performance (ROWS, index, spool) → example (balance, threshold crossing).

**Likely follow-ups:**
- *Why does LAST_VALUE return the current row?* → default frame ends at current row.
- *ROWS vs RANGE?* → physical offset vs value peers; RANGE slower, ties share a result.
- *Year-to-date and quarter-to-date together?* → two SUMs with different PARTITION BY (year / year, quarter) and same ORDER BY.
- *Rows where cumulative sum first exceeds N?* → filter run > N, ROW_NUMBER = 1.

**Signals interviewers look for:** explicit frames, tie awareness, performance awareness (spool).

### 11. Check yourself
- [ ] Predict the output of LAST_VALUE with and without an explicit frame on a 4-row partition.
- [ ] Write a running total that resets each month.
- [ ] Show the `Worktable` reads difference between RANGE and ROWS.
- [ ] Compute running total and remaining balance in one query.
<!-- explainer:end -->

## Practice questions

- [Basic] [How do you compute a running total of daily sales ordered by date? Write the query.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44168_How_do_you_compute_a_running_total_of_daily_sales_ordered_by.md>)
- [Intermediate] [How would you find the running maximum value seen so far in an ordered set using MAX() OVER?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44398_How_would_you_find_the_running_maximum_value_seen_so_far_in_.md>)
- [Intermediate] [How would you compute the Nth occurrence of an event per user (e.g. their 3rd purchase) and its timestamp using ROW_NUMBER or NTH_VALUE?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44101_How_would_you_compute_the_Nth_occurrence_of_an_event_per_use.md>)
- [Intermediate] [You need each customer's first purchase date shown on every one of their order rows; which window function and frame would you use?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44191_You_need_each_customer_s_first_purchase_date_shown_on_every_.md>)
- [Intermediate] [Write a query to compute both the running total and the grand total in the same result set.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44224_Write_a_query_to_compute_both_the_running_total_and_the_gran.md>)
- [Intermediate] [How do you compute a cumulative count of orders per customer that continues growing across dates (partitioned running COUNT)?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44253_How_do_you_compute_a_cumulative_count_of_orders_per_customer.md>)
- [Deep] [Compare computing deciles with NTILE(10) versus WIDTH_BUCKET / PERCENT_RANK; when does NTILE give unequal buckets and does that matter for reporting?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44102_Compare_computing_deciles_with_NTILE_10_versus_WIDTH_BUCKET_.md>)
- [Deep] [How would you compute both a per-store daily rank and an all-time rank of sales in one pass and prove that the two OVER clauses can share a single sort?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44322_How_would_you_compute_both_a_per-store_daily_rank_and_an_all.md>)

## Related concepts

- [Aggregation Query Practice](<../SQL_Fundamentals/Aggregation_Query_Practice.md>) *(SQL Fundamentals)*
- [Classic SQL Query Problems](<../../Database_Internals/DBMS_Concepts/Classic_SQL_Query_Problems.md>) *(DBMS Concepts)*
- [Window Functions Overview](<../SQL_Functions/Window_Functions_Overview.md>) *(SQL Functions)*
- [Advanced Aggregation](<../Aggregation/Advanced_Aggregation.md>) *(Aggregation)*

## References

- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Subqueries
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Table expressions
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — T-SQL for data analysis
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Dividing Data with NTILE
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Solving Queries with Ranking Functions › Finding the First N Rows of Every Group
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Solving Queries with Ranking Functions › Solving the Bonus Problem
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Using Window Aggregates to Solve Common Queries
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Discovering Windowing Functions › Calculating Running Totals
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Discovering Windowing Functions › Solutions to Exercise 8-2: Summarizing Results with Window Aggregates
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Thinking About Performance › Solutions to Exercise 12-1: Variables
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 18 Row-By-Row Processing From Cursors And Other Causes
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Data Models › Multiple calculations
- *Databricks Certified Associate Developer for Apache Spark* — Saba Shah — Advanced Spark SQL operations › Calculating cumulative sum using window functions
- *SQL Server Tutorial* (YouTube series) — Part 112: Calculate running total in SQL Server 2012 ([watch](https://www.youtube.com/watch?v=BMzZmq12YJI))
- *SQL Server Tutorial* (YouTube series) — Part 116: Difference between rows and range ([watch](https://www.youtube.com/watch?v=hM1YKSxltDU))

