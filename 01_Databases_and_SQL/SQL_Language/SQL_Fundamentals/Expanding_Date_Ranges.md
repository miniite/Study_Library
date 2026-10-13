---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Expanding_Date_Ranges"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: []
---

# Expanding Date Ranges

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** To turn ranges (`start_date`…`end_date`) into one row per day — or to make days with no activity show up as zeros — join to a **calendar (date dimension) table**; it replaces loops and per-row date arithmetic with a single, indexable set-based join.

### 1. The big picture
Facts are usually stored *sparsely*: an order exists on a day only if something happened; a subscription is one row with a start and end. Reporting wants the opposite — *dense* output: "active subscriptions per day", "sales per day including zeros", "revenue by fiscal quarter". Bridging sparse storage and dense reporting is the job of a **calendar table** (a `dim_date`): one row per date with pre-computed attributes (year, quarter, fiscal period, weekday, holiday flag).

Two shapes recur:
- **Gap filling** — `calendar LEFT JOIN facts` so missing days appear as 0.
- **Range expansion** — `ranges JOIN calendar ON date BETWEEN start AND end` to turn one row into N daily rows (or to count how many ranges cover each day).

The same family includes pivoting rows to columns for reporting, and partitioning big date-keyed tables so date predicates prune data.

### 2. Core concepts and vocabulary
- **Calendar / date dimension** — a table with one row per day and descriptive attributes.
- **Gap filling** — adding rows for periods that have no data.
- **Range join / non-equi join** — a join using `BETWEEN` or inequalities, not equality.
- **Half-open interval** — `[start, end)`; start inclusive, end exclusive; avoids off-by-one and midnight issues.
- **Tally / numbers table** — a table of integers 1..N used to generate sequences.
- **Recursive CTE** — a self-referencing CTE; can generate a date series but is row-by-row and capped by `MAXRECURSION`.
- **Pivot** — rows → columns.
- **Partition pruning** — the optimizer skipping partitions that cannot match the predicate.

### 3. How it works, step by step
**Build the calendar once:**
1. Generate a contiguous date series (tally table, recursive CTE, or `GENERATE_SERIES`).
2. Add attributes with built-in functions (`YEAR`, `DATEPART`, `EOMONTH`) and lookups (fiscal calendar, holidays).
3. Make `date` the clustered primary key; add indexes for common filters (`year, month`).

**Gap fill:** `SELECT c.date, COALESCE(SUM(f.amount),0) FROM calendar c LEFT JOIN fact f ON f.date = c.date WHERE c.date BETWEEN … GROUP BY c.date`. The `WHERE` filter must be on the *calendar* side; a filter on `f` in `WHERE` turns the outer join into an inner join.

**Range expansion:** `FROM subscriptions s JOIN calendar c ON c.date >= s.start_date AND c.date <= COALESCE(s.end_date, @as_of)`; aggregate with `COUNT(*) GROUP BY c.date`.

### 4. Worked example
```sql
-- 1) Build a calendar for 2020-01-01 .. 2035-12-31 with a tally CTE
DECLARE @start date = '2020-01-01', @end date = '2035-12-31';
WITH n AS (
    SELECT TOP (DATEDIFF(DAY,@start,@end)+1)
           ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) - 1 AS i
    FROM sys.all_objects a CROSS JOIN sys.all_objects b
)
SELECT DATEADD(DAY, i, @start)                                   AS d,
       YEAR(DATEADD(DAY,i,@start))                               AS yr,
       DATEPART(QUARTER, DATEADD(DAY,i,@start))                  AS qtr,
       DATENAME(WEEKDAY, DATEADD(DAY,i,@start))                  AS weekday_name,
       CASE WHEN DATEPART(WEEKDAY, DATEADD(DAY,i,@start)) IN (1,7) THEN 1 ELSE 0 END AS is_weekend
INTO dbo.dim_date
FROM n;
ALTER TABLE dbo.dim_date ALTER COLUMN d date NOT NULL;
ALTER TABLE dbo.dim_date ADD CONSTRAINT PK_dim_date PRIMARY KEY CLUSTERED (d);

-- 2) Daily sales including zero days
SELECT c.d, COALESCE(SUM(s.amount), 0) AS sales
FROM   dbo.dim_date c
LEFT JOIN dbo.sales s ON s.sale_date = c.d
WHERE  c.d >= '2026-09-01' AND c.d < '2026-10-01'
GROUP  BY c.d
ORDER  BY c.d;

-- 3) Active subscriptions per day (ranges → days)
SELECT c.d, COUNT(*) AS active_subs
FROM   dbo.subscriptions s
JOIN   dbo.dim_date c
       ON c.d >= s.start_date
      AND c.d <= COALESCE(s.end_date, '2026-10-31')
WHERE  c.d >= '2026-09-01' AND c.d < '2026-11-01'
GROUP  BY c.d;

-- 4) Pivot a long event-type table into one column per type
SELECT user_id,
       SUM(CASE WHEN event_type='view'     THEN 1 ELSE 0 END) AS views,
       SUM(CASE WHEN event_type='cart'     THEN 1 ELSE 0 END) AS carts,
       SUM(CASE WHEN event_type='purchase' THEN 1 ELSE 0 END) AS purchases
FROM   dbo.events GROUP BY user_id;
```
Reading (3): each subscription row is multiplied by the number of days it covers *within the reporting window*; the `WHERE` on the calendar bounds the output. Cost ≈ (subscriptions × days covered), so restrict the window.

### 5. Under the hood (optional at L4)
A non-equi join cannot use a hash join; the optimizer chooses nested loops (seek into the calendar's clustered index for each subscription over the range) or a merge join on a derived range. Because the calendar's key is `d`, each probe is an **index range seek** `d >= start AND d <= end` — efficient. A recursive CTE series is evaluated iteratively through a spool, each recursion level adding one row; `OPTION (MAXRECURSION 0)` is needed beyond 100 levels, and it is much slower than a tally table. **Partitioning** a large table by month stores each month in its own B-tree/heap; a `WHERE event_date >= '2026-09-01' AND event_date < '2026-10-01'` lets the engine eliminate the other partitions (pruning), but only when the predicate is on the partition column in a sargable form. A unique index on a partitioned table must include the partition column, which is the "cross-partition unique key" pitfall.

### 6. Performance and design implications
- A persisted calendar costs a few MB and is reused by everything; computing fiscal logic inline (`CASE` on month) repeats work and drifts between reports.
- Make `d` the clustered PK; keep the data type `date`, not `datetime`, to avoid time-of-day misses.
- Use half-open ranges `ts >= @d AND ts < DATEADD(DAY,1,@d)` for datetime columns.
- Always bound range expansions by a reporting window; unbounded `end_date IS NULL` ranges explode.
- For large partitioned fact tables: partition by month, align indexes, switch old partitions out (`SWITCH`) instead of `DELETE`.
- Check the plan for `Actual Partitions Accessed` to confirm pruning.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `GENERATE_SERIES(start, stop)` (SQL Server 2022+, PG) | Number/date series | Build calendar |
| Recursive CTE + `OPTION (MAXRECURSION 0)` | Series fallback | Small ranges |
| `EOMONTH`, `DATEFROMPARTS`, `DATETRUNC` (2022+) | Date arithmetic | Calendar attributes |
| `PIVOT` / conditional aggregation | Rows → columns | Wide reports |
| `CREATE PARTITION FUNCTION … RANGE RIGHT` | Date partitioning | Large time-series tables |
| `ALTER TABLE … SWITCH PARTITION` | Metadata-only move | Archive/purge |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Calendar table | One-off ad-hoc query where a short recursive CTE suffices |
| Gap fill via `LEFT JOIN` | Data already dense |
| Range expansion join | Ranges are very long and numerous — consider interval math (sum of +1/−1 deltas with running total) |
| Partition by date | Table < ~100M rows — indexing alone may do |

### 9. Common mistakes
- `WHERE f.date >= …` after a `LEFT JOIN` → outer join becomes inner → zero days vanish → filter on calendar column.
- `BETWEEN` on datetime columns → misses last day's times → half-open range.
- Recursive CTE for 20 years → slow/`MAXRECURSION` error → tally table.
- Counting `COUNT(*)` on a left-joined zero day → returns 1 → `COUNT(f.id)`.
- Holiday/fiscal logic copied into many reports → inconsistent → one calendar.
- Unique key on partitioned table without the partition column → creation fails.
- Open-ended ranges expanded to "today" at run time → results change silently → pass an explicit `@as_of`.

### 10. 🎯 Interview kit
**30-second answer:** "I keep a calendar dimension with one row per date and its attributes. To show zero days I `LEFT JOIN` facts to it; to expand ranges I `JOIN` on `date BETWEEN start AND end`; and I bound the window so the join is an index range seek."

**Structure a longer answer:** clarify (grain, time zone, open-ended ranges) → mechanism (calendar, gap fill, range join) → trade-offs (calendar vs recursive CTE vs interval deltas) → scenario (active subs per day for a month).

**Likely follow-ups:**
- *Why a calendar over `YEAR()` and `CASE`?* → one source of truth, indexable, holiday/fiscal support.
- *Pitfall of the left join?* → filter placement; `COUNT(col)`.
- *Partition pruning?* → predicate on partition column, sargable.
- *Cross-partition uniqueness?* → unique index must include the partition key.

**Signals interviewers look for:** half-open ranges, outer-join semantics, bounded windows.

### 11. Check yourself
- [ ] Build a calendar and give it a clustered PK.
- [ ] Produce daily sales with zeros for a month.
- [ ] Expand subscription ranges into active-per-day counts.
- [ ] Explain why the `WHERE` placement breaks the outer join.
- [ ] Explain partition pruning and the unique-key limitation.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [How would you write a query to pivot a narrow key-value table (attribute, value) into a wide table for reporting?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40014_How_would_you_write_a_query_to_pivot_a_narrow_key-value_tabl.md>)
- [Intermediate] [What is a date dimension (calendar table), and how would you generate one in SQL to support time-based reporting?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40016_What_is_a_date_dimension_calendar_table_and_how_would_you_ge.md>)
- [Intermediate] [How do you write a query to fill gaps in a daily sales series so days with zero sales still appear, using a calendar table?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40018_How_do_you_write_a_query_to_fill_gaps_in_a_daily_sales_serie.md>)
- [Intermediate] [What is table partitioning by time, and why does it help a fast-growing metrics table stay queryable?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-41021_What_is_table_partitioning_by_time_and_why_does_it_help_a_fa.md>)
- [Intermediate] [Explain how you'd generate and use a date/calendar dimension with fiscal-quarter and holiday flags, and why joining facts to a calendar table beats computing fiscal periods inline on every query.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40195_Explain_how_you_d_generate_and_use_a_date_calendar_dimension.md>)
- [Deep] [How do you declaratively range-partition a fast-growing events table by month? Explain partition pruning, the cross-partition unique-key pitfall, and routine partition maintenance.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39895_How_do_you_declaratively_range-partition_a_fast-growing_even.md>)
- [Deep] [Write a query to pivot rows into columns (e.g. monthly sales per product).](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40675_Write_a_query_to_pivot_rows_into_columns_e_g_monthly_sales_p.md>)
- [Deep] [Explain how you'd expand a range table of (subscription_id, start_date, end_date) into daily active rows in a warehouse using a generated date sequence, and how to avoid materializing a huge cross product.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40131_Explain_how_you_d_expand_a_range_table_of_subscription_id_st.md>)

## Related concepts

- [Classic SQL Query Problems](<../../Database_Internals/DBMS_Concepts/Classic_SQL_Query_Problems.md>) *(DBMS Concepts)*
- [Advanced Aggregation](<../Aggregation/Advanced_Aggregation.md>) *(Aggregation)*
- [UDF Performance](<../SQL_Functions/UDF_Performance.md>) *(SQL Functions)*

## References

- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Spark SQL and DataFrames Interacting with External Data Sources › Pivoting
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Putting It All Together › Period-to-Date Calculations
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Moving Totals and Simple Moving Averages
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — 13. Implementing Logic in the Database › Defining Automatically Populated Columns
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Writing Advanced Queries › Pivoted Queries
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Writing Advanced Queries › Using the PIVOT Function
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Where to Go Next › Organizing Your Notebooks
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Programmable objects
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › PIVOT and UNPIVOT Transformations
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › Understanding PIVOT
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › Best Practices for PIVOT and UNPIVOT
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Data Transformation and Enrichment Patterns › 8. Data Transformation and Enrichment Patterns

