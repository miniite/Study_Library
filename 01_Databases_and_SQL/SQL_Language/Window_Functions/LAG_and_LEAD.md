---
id: "01_Databases_and_SQL/SQL_Language/Window_Functions/LAG_and_LEAD"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/Window_Functions"
tools: []
---

# LAG and LEAD

**L4 Advanced** · [↑ Window Functions](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** LAG and LEAD let a row read a column from the row N positions before or after it in the window order — no self-join needed.

### 1. The big picture
Comparing a row to its neighbour is the heart of time-series and change analysis: month-over-month growth, time since the previous order, "did the status change?", gaps between events. Before 2012 (SQL Server) it needed a self-join on `rownum = rownum - 1` or a correlated subquery. LAG (look back) and LEAD (look ahead) do it in one pass over sorted data.

They are *offset* functions: they ignore frames and use only PARTITION BY and ORDER BY. Their sibling functions FIRST_VALUE/LAST_VALUE (frame-based) are covered in the running-totals lesson.

### 2. Core concepts and vocabulary
- **LAG(expr [, offset [, default]])** — value from `offset` rows earlier in the ordered partition.
- **LEAD(expr [, offset [, default]])** — value from `offset` rows later.
- **Offset** — how many rows to move; default 1.
- **Default** — returned when there is no such row; default NULL. Needs offset to be given.
- **IGNORE NULLS / RESPECT NULLS** — skip NULL values when looking back/forward (not in SQL Server before 2022; supported by Oracle, Snowflake, BigQuery, Spark).
- **Delta / growth** — `val − LAG(val)`; `val / LAG(val) − 1`.
- **Gap** — missing period in the data; LAG counts *rows*, not days.

### 3. How it works, step by step
1. Partition rows (e.g. per customer or per ticker) so one entity never reads another's rows.
2. Sort each partition by the window ORDER BY (required).
3. For the current row at position p, return the value at p − offset (LAG) or p + offset (LEAD). If out of range, return default or NULL.
4. Use the result in an expression or a filter (via CTE).

Position and result:

| ordermonth | val | LAG(val) | LEAD(val) | LAG(val,2,0) |
|---|---|---|---|---|
| 2024-01 | 100 | NULL | 120 | 0 |
| 2024-02 | 120 | 100 | 90 | 0 |
| 2024-03 | 90 | 120 | NULL | 100 |

### 4. Worked example
```sql
-- month-over-month change and growth, safe against NULL and zero
WITH m AS (
  SELECT custid, ordermonth, val,
         LAG(val) OVER (PARTITION BY custid ORDER BY ordermonth) AS prev_val
  FROM Sales.EmpOrders)
SELECT custid, ordermonth, val, prev_val,
       val - prev_val                                        AS delta,
       (val - prev_val) * 1.0 / NULLIF(prev_val, 0)          AS growth_rate
FROM m;

-- same month last year on a *complete* monthly series: offset 12
SELECT ym, revenue, revenue - LAG(revenue, 12) OVER (ORDER BY ym) AS yoy_delta
FROM dbo.MonthlyRevenue;

-- change detection in an ETL feed
SELECT *, CASE WHEN status <> LAG(status) OVER (PARTITION BY id ORDER BY ts)
               OR LAG(status) OVER (PARTITION BY id ORDER BY ts) IS NULL
          THEN 1 ELSE 0 END AS changed
FROM dbo.StatusHistory;

-- last order per customer without an anti-join: LEAD is NULL
SELECT * FROM (
  SELECT *, LEAD(orderid) OVER (PARTITION BY custid ORDER BY orderdate, orderid) AS next_id
  FROM Sales.Orders) x
WHERE next_id IS NULL;
```
Time to next status change: `DATEDIFF(minute, ts, LEAD(ts) OVER (PARTITION BY order_id ORDER BY ts))`.

Forward-fill a sparse column where IGNORE NULLS isn't available:
```sql
WITH g AS (
  SELECT *, COUNT(price) OVER (PARTITION BY sku ORDER BY d ROWS UNBOUNDED PRECEDING) AS grp
  FROM dbo.Prices)
SELECT *, MAX(price) OVER (PARTITION BY sku, grp) AS price_filled FROM g;
```
(`COUNT(price)` only increments on non-NULLs, so each NULL run shares a group with its last real value.)

### 5. Under the hood
- The plan shape is the same as ranking: Segment on partition, Sort on `(partition, order)`, then a **Window Spool/Sequence Project** that keeps a small buffer of rows so it can serve the offset. Cost ≈ sort + one pass; the buffer is about `offset` rows long.
- Large offsets (LAG(x, 1000)) enlarge the buffer and, in SQL Server, may require spooling.
- LAG ignores any frame clause; specifying `ROWS` is a syntax error for LAG/LEAD.
- Because LAG counts **rows**, the meaning of `LAG(x, 12)` as "12 months ago" holds only if the series has exactly one row per month. Missing months silently shift everything.
- NULL data values are returned as-is (RESPECT NULLS default); with IGNORE NULLS the engine scans back to the nearest non-NULL, which can be O(n) per row in the worst case.

### 6. Performance and design implications
- Index `(partition cols, order col) INCLUDE (value cols)` to avoid the sort.
- Build the complete calendar (cross join keys × dates, LEFT JOIN facts) before LAG when "same period last year" must be time-correct; or join on `date = DATEADD(year, -1, date)` instead.
- Several LAG/LEAD calls with the same OVER spec share one sort; keep specs identical.
- Prefer LAG over self-joins: usually one scan versus two plus a join; 3–10× fewer reads.
- Guard divisions: `NULLIF(prev, 0)`; NULL propagates, so the first row's growth is NULL — decide whether to filter or default it.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `LAG(x)` / `LEAD(x)` | Previous / next row's value | Deltas, change detection |
| `LAG(x, n, d)` | n back, default d | Seasonal comparison, avoid NULL first row |
| `OVER (PARTITION BY k ORDER BY t)` | Per-entity sequence | Always include the partition |
| `IGNORE NULLS` | Skip NULLs | Carry last known value |
| `NULLIF(den, 0)` | Avoid divide-by-zero | Growth rates |
| `DATEDIFF(...)` with LEAD | Duration to next event | Status durations, sessions |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Rows are an ordered, gap-free sequence | The series has gaps → pad dates or join on date arithmetic |
| One neighbour needed | Many neighbours / variable window → frame aggregates |
| You need first/last of group | Use FIRST_VALUE/LAST_VALUE, not LAG with huge offset |

### 9. Common mistakes
- Omitting PARTITION BY → one entity's last row becomes the next entity's "previous" → partition by the entity key.
- Non-unique ORDER BY → neighbour is arbitrary on ties → add a tiebreaker.
- `LAG(x, 12)` on a series with a missing month → compares wrong month → complete the calendar.
- Dividing by LAG without guarding zero/NULL → divide-by-zero or NULL → NULLIF and decide first-row handling.
- Specifying a default without offset → syntax error → `LAG(x, 1, 0)`.
- NULLs in ORDER BY column placed first/last differently across engines → different neighbours → NULLS FIRST/LAST or COALESCE.
- Descending ORDER BY flips meaning: LAG then returns the *later* row → know your direction.

### 10. 🎯 Interview kit
**30-second answer:** LAG returns a value from a previous row in the ordered partition, LEAD from a following row; optional offset (default 1) and default value (default NULL). They replace self-joins for comparisons between adjacent rows.

**Structure a longer answer:** clarify partition/order → syntax with offset/default → use case (delta, change flag) → gaps and ties → index/sort cost.

**Likely follow-ups:**
- *First row of a partition?* → NULL or the default you supplied.
- *Month-over-month growth safely?* → `(v − LAG)/NULLIF(LAG,0)`.
- *Sessionization?* → LAG(ts), flag gaps > 30 min, cumulative SUM of flags.
- *Previous non-null value?* → LAG IGNORE NULLS or the COUNT/MAX grouping trick.
- *What breaks LAG(x,12) for YoY?* → missing periods.

**Signals interviewers look for:** partitioning discipline, awareness that offsets are row-based, null-safe maths.

### 11. Check yourself
- [ ] Write month-over-month % change with a zero guard.
- [ ] Explain why LAG(x,12) can lie and fix it with a calendar table.
- [ ] Flag status changes in a history table using LAG.
- [ ] Forward-fill a column without IGNORE NULLS.
<!-- explainer:end -->

## Practice questions

- [Basic] [What are LAG() and LEAD() used for? Give a simple example.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44044_What_are_LAG_and_LEAD_used_for_Give_a_simple_example.md>)
- [Basic] [Explain what LAG(sales, 1, 0) returns for the first row of a partition.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44180_Explain_what_LAG_sales_1_0_returns_for_the_first_row_of_a_pa.md>)
- [Intermediate] [What does the second and third argument of LAG() (offset and default) do?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44066_What_does_the_second_and_third_argument_of_LAG_offset_and_de.md>)
- [Intermediate] [How would you compute a growth-rate column that safely handles a zero or null previous value from LAG without dividing by zero?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44151_How_would_you_compute_a_growth-rate_column_that_safely_handl.md>)
- [Intermediate] [How would you compute a month-over-month percentage growth using LAG and guard against divide-by-zero?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44209_How_would_you_compute_a_month-over-month_percentage_growth_u.md>)
- [Intermediate] [Write a query to forward-fill a null status using the last non-null value with LAG IGNORE NULLS (or an equivalent).](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44236_Write_a_query_to_forward-fill_a_null_status_using_the_last_n.md>)
- [Deep] [When computing FIRST_VALUE/LAST_VALUE, why can specifying IGNORE NULLS change both correctness and performance, and which engines support it?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44065_When_computing_FIRST_VALUE_LAST_VALUE_why_can_specifying_IGN.md>)
- [Deep] [How would you compute a 'previous non-null price' where many rows have null prices, using LAG with IGNORE NULLS or a framed last-value fallback?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44147_How_would_you_compute_a_previous_non-null_price_where_many_r.md>)

## Related concepts

- [Gaps and Islands](<../SQL_Fundamentals/Gaps_and_Islands.md>) *(SQL Fundamentals)*
- [Window Functions Overview](<../SQL_Functions/Window_Functions_Overview.md>) *(SQL Functions)*
- [UDF Performance](<../SQL_Functions/UDF_Performance.md>) *(SQL Functions)*

## References

- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Understanding LAG and LEAD
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Understanding FIRSTVALUE and LASTVALUE
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Using the Offset Functions to Solve Queries
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Comparing Performance
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — T-SQL for data analysis
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Discovering Windowing Functions › FIRSTVALUE and LASTVALUE
- *SQL Server Tutorial* (YouTube series) — Part 114: Lead and Lag functions in SQL Server 2012 ([watch](https://www.youtube.com/watch?v=l_Zn5sdkamM))

