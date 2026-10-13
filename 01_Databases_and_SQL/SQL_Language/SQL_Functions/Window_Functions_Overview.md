---
id: "01_Databases_and_SQL/SQL_Language/SQL_Functions/Window_Functions_Overview"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Functions"
tools: []
---

# Window Functions Overview

**L4 Advanced** · [↑ SQL Functions](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A window function computes a value for *each row* from a related set of rows (its "window") defined by `OVER (PARTITION BY … ORDER BY … frame)` — so you get rankings, running totals, neighbours and percentages *without collapsing rows* the way `GROUP BY` does.

### 1. The big picture
`GROUP BY` answers "one answer per group" and throws away the detail rows. Subqueries can keep the detail but are verbose and slow (each correlated subquery re-reads data). Window functions solve the awkward middle: *"for every order, show the order and the customer's total / the previous order / its rank within the customer."*

Itzik Ben-Gan defines it precisely: a window function "for each row, computes a scalar result value based on a calculation against a subset of the rows from the underlying query" — the subset is the window, described by the `OVER` clause. They are one of the most heavily tested SQL topics because they replace self-joins, cursors and correlated subqueries with a single pass.

Four families:

| Family | Functions | Question answered |
|---|---|---|
| **Ranking** | `ROW_NUMBER`, `RANK`, `DENSE_RANK`, `NTILE` | "What position/bucket is this row in?" |
| **Aggregate** | `SUM`, `AVG`, `COUNT`, `MIN`, `MAX` `OVER (…)` | "What's the total/average around this row?" |
| **Offset** | `LAG`, `LEAD`, `FIRST_VALUE`, `LAST_VALUE`, `NTH_VALUE` | "What is the value in another row?" |
| **Distribution** | `PERCENT_RANK`, `CUME_DIST`, `PERCENTILE_CONT/DISC` | "Where does this row sit in the distribution?" |

### 2. Core concepts and vocabulary
- **Window** — the set of rows a function looks at for the current row.
- **`OVER()`** — the clause that turns a function into a window function.
- **`PARTITION BY`** — splits rows into independent groups; the window never crosses a partition. Omitted → the whole result is one partition.
- **`ORDER BY` (inside OVER)** — defines order within the partition (required for ranking, offset functions, running totals).
- **Frame** — a sub-range of the partition relative to the current row: `ROWS`/`RANGE`/`GROUPS BETWEEN <start> AND <end>`.
- **Frame bounds** — `UNBOUNDED PRECEDING`, `n PRECEDING`, `CURRENT ROW`, `n FOLLOWING`, `UNBOUNDED FOLLOWING`.
- **Peer rows** — rows with the same `ORDER BY` value; `RANGE` treats peers as one unit, `ROWS` doesn't.
- **Ordered-set aggregate** — `PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY x)`: the ordering is part of the aggregate itself.
- **Logical order of evaluation** — window functions run after `WHERE`, `GROUP BY` and `HAVING`, before `SELECT DISTINCT` and `ORDER BY`; so you can only use them in `SELECT` and `ORDER BY`, not directly in `WHERE`.

### 3. How it works, step by step
For `SUM(amount) OVER (PARTITION BY region ORDER BY order_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)`:

1. The query's `FROM/WHERE/GROUP BY/HAVING` run first, producing a row set.
2. The engine **partitions** it by `region`.
3. Within each partition it **sorts** by `order_date`.
4. For each row it builds the **frame**: from the first row of the partition to this row.
5. It computes `SUM` over that frame and attaches it to the row.
6. The final `SELECT`/`ORDER BY` proceed; row count is unchanged.

**Examples of each family (T-SQL / standard):**

```sql
-- Aggregate window: keep detail, add region totals
SELECT Region, OrderID, Amount,
       SUM(Amount) OVER (PARTITION BY Region) AS TotalRegionSales,
       Amount * 100.0 / SUM(Amount) OVER (PARTITION BY Region) AS pct_of_region
FROM Orders;

-- Ranking
SELECT CustomerID, OrderDate,
       ROW_NUMBER() OVER (PARTITION BY CustomerID ORDER BY OrderDate) AS rn
FROM Orders;

-- Offset
SELECT EmployeeID, OrderDate,
       LAG(OrderDate)  OVER (PARTITION BY EmployeeID ORDER BY OrderDate) AS PrevOrder,
       LEAD(OrderDate) OVER (PARTITION BY EmployeeID ORDER BY OrderDate) AS NextOrder
FROM Orders;
```

Ranking functions at a glance (scores 90, 80, 80, 70):

| Score | ROW_NUMBER | RANK | DENSE_RANK |
|---|---|---|---|
| 90 | 1 | 1 | 1 |
| 80 | 2 | 2 | 2 |
| 80 | 3 | 2 | 2 |
| 70 | 4 | 4 | 3 |

`ROW_NUMBER` is unique (ties broken arbitrarily unless you add a tie-breaker), `RANK` leaves gaps after ties, `DENSE_RANK` doesn't. `NTILE(n)` splits into n buckets; with 830 rows and `NTILE(10)` each tile gets 83. When rows don't divide evenly, the first tiles each receive one extra row (e.g. 102 rows into 5 tiles → 21,21,20,20,20).

### 4. Worked example
**Leaderboard limited to exactly the top 10, deduplicate to latest per key, and running total:**

```sql
-- Exactly 10 rows even with ties → ROW_NUMBER (add a deterministic tie-breaker)
SELECT *
FROM (SELECT player, score,
             ROW_NUMBER() OVER (ORDER BY score DESC, player) AS rn
      FROM leaderboard) x
WHERE rn <= 10;

-- All people tied at 10th place included → RANK (can return > 10 rows)
-- No gaps in position numbers shown to users → DENSE_RANK

-- Latest row per customer (replaces GROUP BY + join back)
WITH ranked AS (
  SELECT *, ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY updated_at DESC) AS rn
  FROM customer_versions)
SELECT * FROM ranked WHERE rn = 1;

-- Running total (explicit frame!)
SELECT order_date, amount,
       SUM(amount) OVER (ORDER BY order_date, order_id
                         ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total
FROM orders;

-- Difference from group average in one pass (replaces correlated subquery)
SELECT product_id, region, sales_amount,
       sales_amount - AVG(sales_amount) OVER (PARTITION BY region) AS diff_from_region_avg
FROM sales;

-- Median
SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) FROM employees;  -- interpolates
SELECT PERCENTILE_DISC(0.5) WITHIN GROUP (ORDER BY salary) FROM employees;  -- an actual value
```

**Reading the "RANGE vs ROWS" gotcha.** With `ORDER BY order_date` and *no explicit frame*, the default frame is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`. If three orders share the same date, all three rows get the *same* running total (the sum through the last peer), not a row-by-row accumulation. Add a unique tie-breaker to the order, or specify `ROWS`.

**The `LAST_VALUE` gotcha.** `LAST_VALUE(x) OVER (PARTITION BY k ORDER BY d)` returns the *current* row's value, because the default frame ends at the current row. Use `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` — or use `FIRST_VALUE` with the opposite sort.

**7-day moving average** with gaps in the data: `AVG(v) OVER (ORDER BY d ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)` averages the previous 6 *rows*, which may span more than 7 days if days are missing. Use a calendar table (left join) or `RANGE BETWEEN INTERVAL '6' DAY PRECEDING AND CURRENT ROW` (PostgreSQL, Oracle) to window by value.

### 5. Under the hood
- **Plan shape (SQL Server).** Segment → Sequence Project (ranking) or Window Spool (frame aggregates) after a Sort on `(partition keys, order keys)`. A matching index `(PARTITION cols, ORDER cols) INCLUDE (...)` removes the Sort. The memory grant for the Sort comes from estimated rows; underestimates spill to tempdb.
- **Frames with `RANGE` use a disk-based spool** in SQL Server (very slow), `ROWS` uses an in-memory spool or a fast-path streaming aggregate. Hence the "specify ROWS" advice — it's both more correct (no peer surprises) and 5–10× faster.
- **Statistical functions are convenience.** Kellenberger shows `PERCENT_RANK = (rank − 1)/(N − 1)` and `CUME_DIST = rank/N`; older code computed them with `RANK()` and `COUNT(*) OVER ()`. `PERCENT_RANK` starts at 0, `CUME_DIST` ends at 1 and counts peers inclusively.
- **Ordered-set aggregates.** `PERCENTILE_CONT` needs the whole sorted set, so it's syntactically `WITHIN GROUP (ORDER BY …)`, not a normal aggregate; SQL Server exposes it only as a window (`OVER (PARTITION BY …)`) function.
- **Offset functions beat self-joins.** `LAG`/`LEAD` read neighbouring rows from the same sorted stream; a self-join `ON t2.id = t1.id - 1` needs gapless ids and two scans.
- **Parallelism.** Partitions can be processed in parallel when distribution is balanced; a single huge partition (or no `PARTITION BY`) serializes the sort.
- **Population vs sample statistics.** `STDDEV_POP`/`VAR_POP` divide by N; `STDDEV_SAMP`/`VAR_SAMP` (and plain `STDEV`/`VAR` in T-SQL) divide by N−1 (Bessel's correction). Use *sample* when your rows are a sample of a larger population, *population* when you have all of it.

### 6. Performance and design implications
- Index for the window: `(partition_cols, order_cols) INCLUDE (value_cols)`.
- Narrow the data before windowing (filter in an inner query/CTE); the window runs over everything that survives `WHERE`.
- One sort can serve many functions if they share the same `PARTITION BY/ORDER BY`; different specs each add a sort. Reuse a named `WINDOW w AS (…)` clause for readability (same plan).
- Prefer window functions to correlated subqueries and self-joins: one pass, O(n log n) vs O(n²).
- Window functions can't be used in `WHERE`/`HAVING` — wrap in a CTE/subquery, or use `QUALIFY` in Snowflake/BigQuery/Databricks/DuckDB/Teradata.
- Approximate analytics (`APPROX_PERCENTILE`, HyperLogLog `APPROX_COUNT_DISTINCT`) trade ~1–2% error for much lower memory on dashboards.
- Large partitions increase memory grant; skewed partitions cause one slow worker in distributed engines (Spark) — salt or pre-aggregate.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ROW_NUMBER() OVER (PARTITION BY k ORDER BY …)` | Unique sequence per partition | Top-N per group, dedup |
| `RANK()`, `DENSE_RANK()` | Ranking with ties | Leaderboards |
| `NTILE(n)` | Equal-ish buckets | Deciles/quartiles |
| `LAG(x, n, default)`, `LEAD(...)` | Previous/next row value | Period-over-period change, gaps |
| `SUM/AVG/COUNT … OVER (…)` | Windowed aggregates | Share-of-total, moving averages |
| `FIRST_VALUE`, `LAST_VALUE`, `NTH_VALUE` | Value at frame position | Baselines, "first purchase" |
| `ROWS BETWEEN … AND …` | Physical-row frame | Running totals, moving windows |
| `RANGE BETWEEN …` | Value-based frame (peers included) | Time-interval windows |
| `PERCENT_RANK`, `CUME_DIST`, `PERCENTILE_CONT/DISC` | Distribution | Percentiles, medians |
| `QUALIFY` | Filter on window results (not in all engines) | Snowflake/BigQuery |
| `FILTER (WHERE …)` | Conditional aggregate (PostgreSQL) | Instead of `SUM(CASE…)` |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Window aggregate to keep detail rows | You only need one row per group — `GROUP BY` is cheaper |
| `ROW_NUMBER` for exact top-N / dedup | Ties matter and must all be returned (use `RANK`) |
| `ROWS` frame for running totals | You deliberately want peers grouped (`RANGE`) |
| `PERCENTILE_CONT` for medians of continuous data | Discrete values where an actual member is required (`PERCENTILE_DISC`) |
| `NTILE` for quick buckets | Exact business thresholds (it splits evenly, so equal values may land in different buckets) |
| `LAG/LEAD` | Gaps in time must be treated as zeros — build a calendar first |

### 9. Common mistakes
- No tie-breaker in `ROW_NUMBER` → non-deterministic result between runs → add a unique column to `ORDER BY`.
- Default frame with `ORDER BY` → duplicate running totals and `LAST_VALUE` = current row → explicit `ROWS` frame.
- Putting window function in `WHERE` → error "windowed functions can only appear in SELECT or ORDER BY" → CTE or `QUALIFY`.
- Using `STDDEV_SAMP` on a complete population (or vice versa) → mis-stated variance → choose by data semantic.
- Using `AVG()` of daily averages as the overall average → ignores group size → compute `SUM/COUNT` or weighted.
- `PARTITION BY` on a high-cardinality column plus big `ORDER BY` without index → sort spills → index and filter first.
- Expecting `NTILE` buckets to be equal when rows aren't divisible → first buckets get one extra → document it.

### 10. 🎯 Interview kit
**30-second answer:** "A window function calculates over a set of rows related to the current row — defined by `OVER (PARTITION BY … ORDER BY … frame)` — and returns a value per row without collapsing them, unlike `GROUP BY`. Families: ranking (`ROW_NUMBER`, `RANK`, `DENSE_RANK`, `NTILE`), aggregates, offset (`LAG`, `LEAD`, `FIRST/LAST_VALUE`) and distribution (`PERCENT_RANK`, `CUME_DIST`, `PERCENTILE_*`)."

**Structure a longer answer:** why not `GROUP BY` → anatomy of `OVER` → one example per family → frame gotchas → performance (indexes, sorts, memory grant).

**Likely follow-ups:**
- *`RANK` vs `DENSE_RANK` vs `ROW_NUMBER`?* → gaps vs no gaps vs unique; choose by tie requirement.
- *Running total?* → `SUM() OVER (ORDER BY … ROWS UNBOUNDED PRECEDING)`.
- *`ROWS` vs `RANGE`?* → physical vs logical (peers); RANGE duplicates totals on ties and is slower.
- *Why does `LAST_VALUE` return the current row?* → default frame ends at current row.
- *`CUME_DIST` vs `PERCENT_RANK`?* → rank/N vs (rank−1)/(N−1).
- *Median?* → `PERCENTILE_CONT(0.5) WITHIN GROUP`; why ordered-set syntax.
- *How do you filter by a window result?* → CTE/subquery or `QUALIFY`.
- *7-day moving average with missing days?* → calendar table or value-based `RANGE`.

**Signals interviewers look for:** correct frames, tie-handling awareness, an index/sort discussion, and recognition that window functions replace self-joins and correlated subqueries.

### 11. Check yourself
- [ ] Write top-3 salaries per department, including ties, and explain which ranking function you used and why.
- [ ] Compute each row's percentage of its group total in one pass.
- [ ] Demonstrate (on paper) why a default-frame running total repeats on tie dates.
- [ ] Rewrite a "previous order date" self-join using `LAG`.
- [ ] State `PERCENT_RANK` and `CUME_DIST` formulas for rank 3 of 10 rows.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is the difference between the AVG function and manually dividing SUM by COUNT?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41308_What_is_the_difference_between_the_AVG_function_and_manually.md>)
- [Intermediate] [Compare RANK, DENSE_RANK, and ROW_NUMBER for a leaderboard with ties. Which one do you choose when you must limit to exactly the top 10 and why?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41176_Compare_RANK_DENSE_RANK_and_ROW_NUMBER_for_a_leaderboard_wit.md>)
- [Deep] [What are window functions and how do they differ from regular aggregate functions?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41292_What_are_window_functions_and_how_do_they_differ_from_regula.md>)
- [Deep] [What is the difference between RANK and DENSE_RANK window functions?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41294_What_is_the_difference_between_RANK_and_DENSE_RANK_window_fu.md>)
- [Deep] [What is a correlated aggregate and how would you compute a running total using a window function?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41325_What_is_a_correlated_aggregate_and_how_would_you_compute_a_r.md>)
- [Deep] [What does the NTILE window function do and when would you use it to create buckets?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41342_What_does_the_NTILE_window_function_do_and_when_would_you_us.md>)
- [Deep] [What is the difference between VAR/VARIANCE and STDDEV aggregate functions?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41351_What_is_the_difference_between_VAR_VARIANCE_and_STDDEV_aggre.md>)
- [Deep] [Why is PERCENTILE_CONT an ordered-set aggregate using WITHIN GROUP rather than a normal aggregate, and how does that change its syntax?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41187_Why_is_PERCENTILE_CONT_an_ordered-set_aggregate_using_WITHIN.md>)

## Related concepts

- [First Last per Group and Running Totals](<../Window_Functions/First_Last_per_Group_and_Running_Totals.md>) *(Window Functions)*
- [ROW_NUMBER RANK and DENSE_RANK](<../Window_Functions/ROW_NUMBER_RANK_and_DENSE_RANK.md>) *(Window Functions)*
- [Window Function Fundamentals and Performance](<../Window_Functions/Window_Function_Fundamentals_and_Performance.md>) *(Window Functions)*

## References

- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — T-SQL for data analysis
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Aggregate Window Functions
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Understanding the Logical Difference Between ROWS and RANGE
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Using PERCENTILECONT and PERCENTILEDISC
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Comparing Statistical Functions to Older Methods
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Summary › � R
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Discovering Windowing Functions › Using NTILE
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Discovering Windowing Functions › Using Window Analytic Functions
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 21 Intelligent Query Processing
- *SQL Server Tutorial* (YouTube series) — Part 113: NTILE function in SQL Server ([watch](https://www.youtube.com/watch?v=lgzHoNK4X24))

