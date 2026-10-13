---
id: "01_Databases_and_SQL/SQL_Language/Window_Functions/Time-based_Windows"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/Window_Functions"
tools: []
---

# Time-based Windows

**L4 Advanced** · [↑ Window Functions](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** SQL windows are defined over *rows* (or value ranges), not calendar time, so time-based calculations — moving averages, rolling counts, sessions — are only correct when you account for gaps, ties and the frame type.

### 1. The big picture
"Average of the last 3 months", "failed logins in the last 15 minutes", "30-minute inactivity sessions" all sound like time windows. A window function, however, is *not date-aware*: `ROWS BETWEEN 2 PRECEDING AND CURRENT ROW` means "the two physical rows before this one", whether they are two days or two years earlier. If a month has no data it simply has no row, and the "3-month" average silently averages three *available* months.

There are three ways to get real time semantics: (1) make the series complete (calendar table), (2) use a value-based `RANGE` frame with an interval where the engine supports it, (3) build the logic from LAG and cumulative flags (gaps and islands, sessions). Streaming engines call the same ideas *tumbling, hopping (sliding) and session windows*.

### 2. Core concepts and vocabulary
- **Row frame (ROWS)** — counts neighbouring rows.
- **Range frame (RANGE)** — includes rows whose ORDER BY *value* falls within a distance of the current value (`RANGE BETWEEN INTERVAL '15' MINUTE PRECEDING AND CURRENT ROW` in PostgreSQL 11+, Oracle, Spark, Snowflake; SQL Server supports only UNBOUNDED/CURRENT ROW bounds).
- **Moving (rolling) average** — aggregate over a sliding frame.
- **Calendar / date spine** — a table with one row per period, used to densify sparse data.
- **Gap** — missing period; **island** — consecutive run of present periods.
- **Sessionization** — grouping events separated by less than an inactivity threshold.
- **Tumbling window** — fixed, non-overlapping periods (hourly buckets).
- **Hopping / sliding window** — fixed length, advancing by a smaller step (60 s windows every 30 s).
- **Session window** — closes after a gap of inactivity.
- **Lateness / watermark** — streaming concept: how long to wait for out-of-order events.

### 3. How it works, step by step
**Rolling average done right (batch):**
1. Aggregate to the desired grain (`GROUP BY` month).
2. LEFT JOIN onto a date spine so empty months exist with value 0 or NULL.
3. Apply `AVG(v) OVER (ORDER BY month ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)`.
4. Optionally blank the first n−1 rows where the frame is incomplete (`COUNT(*) OVER (same frame) = 3`).

**Sessionization:**
1. `prev = LAG(ts) OVER (PARTITION BY user ORDER BY ts)`.
2. `is_new = CASE WHEN prev IS NULL OR ts − prev > 30 min THEN 1 ELSE 0 END`.
3. `session_id = SUM(is_new) OVER (PARTITION BY user ORDER BY ts ROWS UNBOUNDED PRECEDING)`.
4. Aggregate by `(user, session_id)`.

### 4. Worked example
```sql
-- 3-month moving average that respects gaps
WITH spine AS (
  SELECT d.month_start, p.product_id
  FROM dbo.DimMonth d CROSS JOIN dbo.DimProduct p),
m AS (
  SELECT s.product_id, s.month_start, COALESCE(SUM(f.amount), 0) AS amt
  FROM spine s
  LEFT JOIN dbo.Sales f
    ON f.product_id = s.product_id
   AND f.sale_date >= s.month_start AND f.sale_date < DATEADD(month, 1, s.month_start)
  GROUP BY s.product_id, s.month_start)
SELECT product_id, month_start, amt,
       AVG(amt) OVER (PARTITION BY product_id ORDER BY month_start
                      ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS avg3
FROM m;

-- sessions, 30-minute inactivity
WITH f AS (
  SELECT user_id, ts,
         CASE WHEN DATEDIFF(minute, LAG(ts) OVER (PARTITION BY user_id ORDER BY ts), ts) > 30
                   OR LAG(ts) OVER (PARTITION BY user_id ORDER BY ts) IS NULL
              THEN 1 ELSE 0 END AS brk
  FROM dbo.Events)
SELECT user_id, ts,
       SUM(brk) OVER (PARTITION BY user_id ORDER BY ts ROWS UNBOUNDED PRECEDING) AS session_id
FROM f;

-- streak of consecutive winning days (gaps-and-islands)
SELECT *, day_no - ROW_NUMBER() OVER (PARTITION BY team ORDER BY day_no) AS island
FROM (SELECT team, day_no FROM dbo.Results WHERE won = 1) w;
```
Count in the last 15 minutes (PostgreSQL-style RANGE):
```sql
COUNT(*) OVER (PARTITION BY account ORDER BY ts
               RANGE BETWEEN INTERVAL '15 minutes' PRECEDING AND CURRENT ROW)
```
In SQL Server, use a correlated `CROSS APPLY (SELECT COUNT(*) ... WHERE ts BETWEEN DATEADD(minute,-15,e.ts) AND e.ts)` or a self-join — slower but time-correct.

### 5. Under the hood
- ROWS frames are physical offsets in the sorted stream; the engine has no concept of "a month" and cannot detect that a row is missing.
- A RANGE-with-interval frame requires the engine to maintain a sliding window over *values*: it advances the start pointer while `ts < current − interval`. Cost is O(n) per partition with sorted input, but needs a spool/buffer sized to the widest window.
- Aggregates that are reversible (SUM, COUNT, AVG) are maintained incrementally (add entering row, subtract leaving row). MIN/MAX/percentiles are not, so they may rescan the frame; PERCENTILE_CONT needs the whole frame sorted — expensive, hence APPROX_PERCENTILE (t-digest / HLL-style sketches) at scale.
- Streaming engines hold window **state** per key until the watermark passes; session windows merge state when a late event bridges two sessions.
- Time zones and DST make "day" ambiguous; bucket on UTC or on a proper calendar dimension.

### 6. Performance and design implications
- Densifying (key × date spine) multiplies rows; restrict the spine to the active range per key.
- Weighted moving averages with recency weights beyond fixed frames need either several LAGs, a join to a weight table, or a lateral/self-join; pure window frames only support equal weights.
- Index `(partition, time)`; keep event timestamps in an indexed, sortable type.
- Pre-aggregate to the smallest meaningful grain before windowing (minute → hour) to cut sort size.
- For exact percentiles over rolling windows, consider approximate sketches and state the error bound you accept (typically 1–2 %).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ROWS BETWEEN n PRECEDING AND CURRENT ROW` | n+1-row sliding frame | Moving average on a dense series |
| `RANGE BETWEEN INTERVAL ... PRECEDING` | Value-based time frame | Rolling counts in engines that support it |
| `LAG(ts)` + `SUM(flag)` | Sessionization | Inactivity gaps |
| `GENERATE_SERIES`, date dimension | Build date spine | Fill gaps |
| `DATE_TRUNC / DATEADD / DATEDIFF` | Bucketing | Tumbling windows |
| `APPROX_PERCENTILE` | Approximate quantile | Large rolling windows |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Dense, regular series → ROWS frame | Sparse series with gaps → densify or use RANGE |
| RANGE-interval available | Engine only supports unbounded ranges (SQL Server) |
| Exact percentile on small frames | Big frames → approximate |
| Window functions for sessionization | Truly unbounded streams → use the stream engine's session windows |

### 9. Common mistakes
- 3-month average via `2 PRECEDING` on sparse data → average of any three rows → date spine or RANGE.
- LAG(x, 12) assuming monthly continuity → wrong comparison → join on date arithmetic.
- Not partitioning by user/account → a session spans different users → partition.
- Using `>` vs `>=` on the inactivity threshold inconsistently → off-by-one sessions → define the rule once.
- Counting the first rows of a moving average as if complete → misleading early values → require full frame.
- Mixing time zones → a "day" has 23/25 hours at DST → use UTC or calendar table.

### 10. 🎯 Interview kit
**30-second answer:** SQL window frames count rows, not time, so a rolling time window is only right if every period has a row. I either build a calendar spine, use a value-based RANGE interval where supported, or construct it with LAG plus cumulative flags for sessions.

**Structure a longer answer:** clarify grain and gaps → ROWS vs RANGE → densify/sessionize → performance (sort, sketches) → streaming equivalent (tumbling/hopping/session windows with watermarks).

**Likely follow-ups:**
- *Day-over-day change in active users?* → LAG on a daily counts table, ensuring no missing days.
- *Sessions with a 30-minute gap?* → LAG + flag + cumulative SUM.
- *Consecutive-day streaks?* → row_number difference (islands).
- *Rolling count in the last 15 minutes in SQL Server?* → APPLY/self-join, no interval RANGE.
- *Exact vs approximate percentile?* → cost vs error.

**Signals interviewers look for:** you notice gaps immediately, you know frame limits of the engine, and you can map batch windows to streaming windows.

### 11. Check yourself
- [ ] Show with sample data how a 3-row frame misleads on a missing month.
- [ ] Write the full sessionization query and explain each CTE.
- [ ] Explain the gaps-and-islands trick `value − ROW_NUMBER()`.
- [ ] Contrast tumbling, hopping and session windows with an example of each.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [How would you find the gap in days between consecutive login dates for each user?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44390_How_would_you_find_the_gap_in_days_between_consecutive_login.md>)
- [Deep] [How would you detect consecutive days of activity (a streak) for a user using LAG or ROW_NUMBER?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44413_How_would_you_detect_consecutive_days_of_activity_a_streak_f.md>)
- [Deep] [How would you calculate month-over-month retention cohorts using LAG across a pivoted cohort table, and where do window functions fit versus a self-join?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44071_How_would_you_calculate_month-over-month_retention_cohorts_u.md>)
- [Deep] [How would you compute a rolling conversion rate (successes over attempts in the last N events) per user using two framed SUMs?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44098_How_would_you_compute_a_rolling_conversion_rate_successes_ov.md>)
- [Deep] [When a window-heavy transformation is materialized as an incremental table, how do you decide the lookback window (e.g. last 3 days) so late-arriving data still gets correct running aggregates?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44285_When_a_window-heavy_transformation_is_materialized_as_an_inc.md>)
- [Deep] [In a fraud pipeline, how would you flag a card whose transaction exceeds N times its trailing 30-day framed average, and why is a RANGE INTERVAL frame preferable to ROWS here?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44306_In_a_fraud_pipeline_how_would_you_flag_a_card_whose_transact.md>)
- [Deep] [Explain how you'd compute business-day-based moving windows (skipping weekends/holidays) using a dense business-day sequence number rather than raw dates.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44327_Explain_how_you_d_compute_business-day-based_moving_windows_.md>)
- [Expert] [How would you use window functions to compute a moving standard deviation, and why can STDDEV OVER be numerically unstable over long frames?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44091_How_would_you_use_window_functions_to_compute_a_moving_stand.md>)

## Related concepts

- [Gaps and Islands](<../SQL_Fundamentals/Gaps_and_Islands.md>) *(SQL Fundamentals)*
- [Classic SQL Query Problems](<../../Database_Internals/DBMS_Concepts/Classic_SQL_Query_Problems.md>) *(DBMS Concepts)*
- [Retention Queries](<../SQL_Fundamentals/Retention_Queries.md>) *(SQL Fundamentals)*
- [Analytical SQL for ML Features](<../SQL_Fundamentals/Analytical_SQL_for_ML_Features.md>) *(SQL Fundamentals)*
- [Binary Search](<../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Binary_Search.md>) *(Data Structures and Algorithms)*

## References

- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Calculating Moving Totals and Averages
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Averages, Moving Averages, and Rate-of-Change
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Summary › � Y, Z
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Queries on Streaming Data › Fixed-time windows

