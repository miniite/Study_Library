---
id: "TOOLS/MySQL/MySQL_Platform/Date_Functions_and_Grouping"
level: 4
type: "concept"
topic: "TOOLS/MySQL/MySQL_Platform"
tools: []
---

# Date Functions and Grouping

**L4 Advanced** · [↑ MySQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** To group by year or month, extract the date part with `YEAR()`, `MONTH()`, `DATE_FORMAT()` or a generated column, `GROUP BY` that expression, and filter with plain date ranges so the index on the date column still works.

### 1. The big picture
"Orders per month", "employees hired per year", "new users per month this year" are the most common reporting queries there are. They all follow one shape: **filter a date range → derive a bucket label from the date → group by the bucket → aggregate**. The only real decisions are *how* to derive the bucket (which function), *where* to filter (on the raw column, not on the derived bucket), and how to handle buckets that have no rows.

This lesson sits next to Date Arithmetic (types and date math) and Ordering and Limits (sorting grouped output); here the focus is extraction and grouping.

### 2. Core concepts and vocabulary
- **Date part extraction** — `YEAR(d)`, `MONTH(d)`, `DAY(d)`/`DAYOFMONTH(d)`, `QUARTER(d)`, `WEEK(d, mode)`, `HOUR(t)`; `EXTRACT(YEAR_MONTH FROM d)` returns e.g. `202610`.
- **`DATE_FORMAT(d, fmt)`** — turns a date into text: `%Y-%m` → `2026-10`, `%Y` year, `%m` zero-padded month, `%b` month abbreviation, `%x-%v` ISO year-week.
- **Bucket / grain** — the unit you group by: year, month, week, day.
- **`GROUP BY`** — collapses rows with equal grouping expressions; aggregates (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`) summarize each group.
- **`ONLY_FULL_GROUP_BY`** — SQL mode (default in 5.7+/8.0) requiring every selected non-aggregated expression to be functionally dependent on the grouping.
- **Sargable predicate** — a condition the optimizer can answer with an index range scan (`col >= x AND col < y`).
- **Generated column** — a computed column (`VIRTUAL`/`STORED`) that can be indexed, e.g. `signup_month`.

### 3. How it works, step by step
1. Decide the **grain** (month).
2. Decide the **range** (this year) and express it on the raw column: `signup_date >= '2026-01-01' AND signup_date < '2027-01-01'`, or computed from the clock: `>= MAKEDATE(YEAR(CURDATE()), 1) AND < MAKEDATE(YEAR(CURDATE()) + 1, 1)`.
3. Build the **bucket expression**: `DATE_FORMAT(signup_date, '%Y-%m')` (text, sorts correctly because it is zero-padded) or `YEAR(d), MONTH(d)` (two numeric columns).
4. **`GROUP BY`** the bucket (repeat the expression, use its alias — MySQL allows aliases in `GROUP BY` — or its position).
5. **Aggregate** and **`ORDER BY`** the bucket so the output is deterministic (grouping does not guarantee order in 8.0).
6. If empty buckets must appear, **left-join from a calendar** of buckets.

### 4. Worked example
```sql
-- Sales per month: label as 'YYYY-MM'
SELECT DATE_FORMAT(order_ts, '%Y-%m') AS ym,
       COUNT(*)                        AS orders,
       SUM(amount)                     AS revenue
FROM   sales
WHERE  order_ts >= '2026-01-01' AND order_ts < '2027-01-01'
GROUP  BY ym
ORDER  BY ym;

-- Year / month / day parts
SELECT YEAR(hire_date)  AS y,
       MONTH(hire_date) AS m,
       DAY(hire_date)   AS d
FROM   employees LIMIT 3;

-- Employees hired per year
SELECT YEAR(hire_date) AS hire_year, COUNT(*) AS hired
FROM   employees
GROUP  BY YEAR(hire_date)
ORDER  BY hire_year;

-- New users per month this year
SELECT MONTH(signup_date) AS month_no, COUNT(*) AS new_users
FROM   users
WHERE  signup_date >= MAKEDATE(YEAR(CURDATE()), 1)
  AND  signup_date <  MAKEDATE(YEAR(CURDATE()) + 1, 1)
GROUP  BY month_no
ORDER  BY month_no;
```
How to read it: the `WHERE` uses the raw column in a half-open range, so with an index on `order_ts` MySQL range-scans only this year's rows; `DATE_FORMAT` is evaluated only for those rows. The month with no signups is **missing** from the last result — add a calendar (see Date Arithmetic) when you need zeros. `MONTH()` alone is only safe because the filter limits the data to a single year; across years you must group by year *and* month, or you merge January 2025 with January 2026.

### 5. Under the hood
- **How MySQL groups.** The optimizer chooses between a *loose/tight index scan* (rows already arrive in group order from an index), a *sort-then-aggregate*, or a *temporary table* keyed by the group expression (`Using temporary` in `EXPLAIN`). Grouping by `DATE_FORMAT(col, …)` cannot use an index for ordering because the function hides the column, so it typically uses a temporary table; grouping by the raw `DATE` column or an indexed generated column can stream.
- **8.0 behaviour.** Before 8.0, `GROUP BY` implicitly sorted its output; in 8.0 it does not, and `GROUP BY col ASC/DESC` was removed. Always add `ORDER BY`.
- **Alias use.** MySQL lets `GROUP BY` and `HAVING` reference select aliases (a MySQL extension); `WHERE` cannot because it runs before `SELECT`.
- **Generated columns.** `ALTER TABLE sales ADD COLUMN order_month DATE GENERATED ALWAYS AS (DATE_FORMAT(order_ts,'%Y-%m-01')) STORED, ADD KEY ix_order_month (order_month);` lets monthly rollups use the index (and covering indexes `(order_month, amount)` make them index-only).
- **Time zones.** `YEAR()`/`MONTH()` operate on whatever clock value you give them; for a `TIMESTAMP` that is the *session-zone* local time, so month boundaries move with `time_zone`.

### 6. Performance and design implications
- **Filter first, derive second:** put the range on the column; derive the bucket afterwards.
- **Avoid `WHERE YEAR(col) = 2026`** — it forces a full scan; use a range.
- **Group by what's cheap:** an integer bucket (`YEAR(d)*100 + MONTH(d)`) groups faster than a formatted string, though both are fine below millions of rows.
- **Pre-aggregate** monthly/daily rollups into a summary table for dashboards on large fact tables.
- **Check with `EXPLAIN`:** `Using temporary; Using filesort` on a large range is a signal to add a generated column + index or a rollup.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `YEAR()`, `MONTH()`, `DAY()`, `QUARTER()`, `WEEK()` | Numeric date parts | Simple grouping |
| `DATE_FORMAT(d,'%Y-%m')` | Text bucket | Readable month labels |
| `EXTRACT(YEAR_MONTH FROM d)` | Numeric `YYYYMM` | Compact sortable bucket |
| `LAST_DAY(d)`, `MAKEDATE(y, n)`, `DATE_SUB(d, INTERVAL DAYOFMONTH(d)-1 DAY)` | Month/year boundaries | Building ranges and month-start values |
| `GROUP BY … WITH ROLLUP` | Adds subtotal and total rows | Year/month totals in one query |
| `sql_mode ONLY_FULL_GROUP_BY` | Rejects ambiguous non-aggregated columns | Keep enabled |
| `default_week_format` | Default `WEEK()` mode | Weekly reports |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `DATE_FORMAT(d,'%Y-%m')` — one readable label | Millions of rows with no supporting index/rollup |
| `YEAR(d), MONTH(d)` — numeric, easy to join to calendars | Forgetting that `MONTH()` alone mixes years |
| Stored generated column + index — hot recurring report | Rarely-run ad-hoc queries (extra write cost) |
| Summary table / materialized rollup — dashboards | Data changes often and freshness must be exact |
| Left join to a calendar — need zero rows | Sparse reports where missing means "no data" is acceptable |

### 9. Common mistakes
- **Grouping by `MONTH()` over multiple years** → months from different years merge → group by year and month.
- **`WHERE YEAR(order_ts) = 2026`** → index unusable → range on raw column.
- **`BETWEEN '2026-01-01' AND '2026-12-31'` on a `DATETIME`** → drops everything after midnight on Dec 31 → `< '2027-01-01'`.
- **Selecting a non-grouped column** → error 1055 under `ONLY_FULL_GROUP_BY` → aggregate it (`MIN`, `ANY_VALUE`) or add it to `GROUP BY`.
- **Relying on `GROUP BY` order** → unspecified in 8.0 → explicit `ORDER BY`.
- **Missing months silently absent** → charts with gaps → calendar left join with `COALESCE`.
- **Text-sorting unpadded labels** (`2026-9` > `2026-10`) → wrong order → zero-padded formats.

### 10. 🎯 Interview kit
**30-second answer:** Filter the date range on the raw column, extract the bucket with `YEAR`/`MONTH`/`DATE_FORMAT`, `GROUP BY` it, aggregate, and `ORDER BY` the bucket. Don't wrap the filter column in a function, group by year and month together, and use a calendar join if empty buckets must show.

**Structure a longer answer:** grain and range → bucket expression → grouping and aggregation → index-friendliness (generated column / rollup) → edge cases (time zones, empty months, multi-year).

**Likely follow-ups:**
- *Why is `WHERE YEAR(col)=2026` slow?* → function on column prevents range scan.
- *How do you include months with zero sales?* → left join a generated month series.
- *What's `ONLY_FULL_GROUP_BY`?* → standard-SQL rule against ambiguous select-list columns.
- *Quarter or ISO-week reporting?* → `QUARTER()`, `YEARWEEK(d, 3)` / `%x-%v`.
- *Totals and subtotals together?* → `WITH ROLLUP`.

**Signals interviewers look for:** correct grain handling, half-open ranges, awareness of index usage, and sorting explicitly.

### 11. Check yourself
- [ ] Write "orders per month in 2026" without a function on the filtered column.
- [ ] Explain why grouping by `MONTH(d)` alone is wrong across years.
- [ ] Add a generated column and index to speed up the monthly rollup, then confirm with `EXPLAIN`.
- [ ] Return zero rows for months with no sales.
- [ ] Explain `Using temporary` in this context and how to remove it.
<!-- explainer:end -->

## Practice questions

- [Basic] [How do you extract the year, month, and day from a date column?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33373_How_do_you_extract_the_year_month_and_day_from_a_date_column.md>)
- [Basic] [How would you extract just the month and year from a DATETIME and group sales by month?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33038_How_would_you_extract_just_the_month_and_year_from_a_DATETIM.md>)
- [Intermediate] [How would you write a query to find the number of employees hired in each year?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33436_How_would_you_write_a_query_to_find_the_number_of_employees_.md>)
- [Intermediate] [Write a query to find the number of new users who signed up each month this year.](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33564_Write_a_query_to_find_the_number_of_new_users_who_signed_up_.md>)

## Related concepts

- [Aggregation Query Practice](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Aggregation_Query_Practice.md>) *(SQL Fundamentals)*
- [Classic SQL Query Problems](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Classic_SQL_Query_Problems.md>) *(DBMS Concepts)*
- [Counting with Joins](<../../../01_Databases_and_SQL/SQL_Language/Aggregation/Counting_with_Joins.md>) *(Aggregation)*

