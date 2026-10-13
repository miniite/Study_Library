---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Gaps_and_Islands"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: []
---

# Gaps and Islands

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** To find runs of consecutive values (streaks, outages, sessions), compute a number that is **constant within each run** — usually `value − ROW_NUMBER()` or `ROW_NUMBER() − ROW_NUMBER() OVER (PARTITION BY flag)` — and `GROUP BY` it; "gaps" are the holes between the runs ("islands").

### 1. The big picture
Many problems are about *continuity*: longest streak of days a metric stayed above a threshold, how long an alert stayed firing, consecutive logins, gaps in an ID sequence, periods when a host was down. The data is a list of rows; the answer is about *runs* of adjacent rows. SQL has no "run" concept, so we manufacture one.

- An **island** is a maximal run of consecutive items.
- A **gap** is the missing stretch between two islands.

Two solution families: the **grouping-key** trick (subtract a row number from the value — the classic Itzik Ben-Gan formulation) and the **LAG/LEAD flag + running sum** approach (mark where a new island starts, then cumulative-sum the markers). Both are set-based and need one sort.

### 2. Core concepts and vocabulary
- **Island** — consecutive values with no break.
- **Gap** — a break in the expected sequence.
- **Island identifier (group key)** — a computed value equal for all rows in the same island.
- **Consecutive** — adjacent in a defined sequence (dates one day apart, integers differing by 1).
- **Row number** — `ROW_NUMBER() OVER (ORDER BY …)`; increases by 1 each row.
- **Delta trick** — if values increase by 1 as row numbers do, `value − rn` is constant inside an island.
- **Start-of-island flag** — 1 when the row isn't adjacent to the previous one.
- **Running sum** — `SUM(flag) OVER (ORDER BY …)`, turning flags into island ids.
- **Flapping** — repeated state changes (firing ↔ resolved).

### 3. How it works, step by step
**Method A — date minus row number (consecutive distinct days):**
1. Make each date unique per entity (`SELECT DISTINCT user_id, activity_date`).
2. Number rows per user ordered by date: `rn`.
3. Compute `grp = DATEADD(DAY, -rn, activity_date)`. Within a streak, date goes up by 1 and `rn` goes up by 1, so `grp` is constant; a missing day makes the date jump by 2 while `rn` increases by 1, so `grp` changes.
4. `GROUP BY user_id, grp`; `MIN(date)` and `MAX(date)` are the island bounds, `COUNT(*)` its length.

**Method B — LAG + running sum (any condition, any spacing):**
1. `prev = LAG(ts) OVER (PARTITION BY k ORDER BY ts)`.
2. `is_new = CASE WHEN prev IS NULL OR DATEDIFF(...) > threshold THEN 1 ELSE 0 END`.
3. `island = SUM(is_new) OVER (PARTITION BY k ORDER BY ts ROWS UNBOUNDED PRECEDING)`.
4. Group by `(k, island)`.

**Method C — two row numbers (islands of a repeating status):** `rn_all − rn_within_status` is constant for each consecutive run of the same status.

**Gaps** — `LEAD(value)`; where `next − value > 1`, a gap exists from `value+1` to `next−1`.

### 4. Worked example
```sql
-- A) Longest streak of days where daily_errors > 100  (threshold run, not mere consecutive days)
WITH flagged AS (
    SELECT day, CASE WHEN daily_errors > 100 THEN 1 ELSE 0 END AS hot
    FROM   dbo.daily_metrics
), numbered AS (
    SELECT day, hot,
           ROW_NUMBER() OVER (ORDER BY day)                       AS rn_all,
           ROW_NUMBER() OVER (PARTITION BY hot ORDER BY day)      AS rn_hot
    FROM   flagged
)
SELECT TOP (1) MIN(day) AS streak_start, MAX(day) AS streak_end, COUNT(*) AS days
FROM   numbered
WHERE  hot = 1
GROUP  BY rn_all - rn_hot
ORDER  BY COUNT(*) DESC;

-- B) Login streaks per user: date - row_number
WITH d AS (SELECT DISTINCT user_id, CAST(login_ts AS date) AS d FROM dbo.logins),
r AS (SELECT user_id, d, ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY d) AS rn FROM d)
SELECT user_id, MIN(d) AS streak_start, MAX(d) AS streak_end, COUNT(*) AS streak_days
FROM   r
GROUP  BY user_id, DATEADD(DAY, -rn, d)
HAVING COUNT(*) >= 3;

-- C) Sessionization: new session after 30 minutes of inactivity
WITH x AS (
    SELECT user_id, ts,
           CASE WHEN DATEDIFF(MINUTE, LAG(ts) OVER (PARTITION BY user_id ORDER BY ts), ts) > 30
                  OR LAG(ts) OVER (PARTITION BY user_id ORDER BY ts) IS NULL
                THEN 1 ELSE 0 END AS new_sess
    FROM dbo.events
), s AS (
    SELECT *, SUM(new_sess) OVER (PARTITION BY user_id ORDER BY ts ROWS UNBOUNDED PRECEDING) AS sess_id
    FROM x
)
SELECT user_id, sess_id, MIN(ts) AS sess_start, MAX(ts) AS sess_end, COUNT(*) AS events
FROM s GROUP BY user_id, sess_id;

-- D) Gaps in an id sequence
SELECT id + 1 AS gap_start, next_id - 1 AS gap_end
FROM  (SELECT id, LEAD(id) OVER (ORDER BY id) AS next_id FROM dbo.invoices) t
WHERE next_id - id > 1;

-- E) Alert flapping: count transitions between firing and resolved
SELECT alert_id, SUM(CASE WHEN state <> prev_state THEN 1 ELSE 0 END) AS transitions
FROM  (SELECT alert_id, state,
              LAG(state) OVER (PARTITION BY alert_id ORDER BY ts) AS prev_state
       FROM dbo.alert_events) t
GROUP BY alert_id;
```
Why (A) differs from a plain consecutive-day streak: days below threshold must *break* the run even though the calendar days are consecutive; the two-row-number method makes `rn_all − rn_hot` constant only across uninterrupted `hot = 1` rows.

### 5. Under the hood (optional at L4)
Both methods are one sort plus a **Segment/Window Spool** pass and a final aggregate. Method A/B rely on the *density* of row numbers: row numbers increase by exactly 1, so any irregularity in the key shows up as a change in the difference. That is why you must dedupe first (two rows on the same day would let `rn` increase while `date` doesn't, and break the invariant). In method C, `ROW_NUMBER() OVER (PARTITION BY hot …)` restarts per status, and the difference of the two counters equals the number of rows of the *other* status seen so far — constant exactly while no other-status row intervenes. For intervals (start,end) rather than points, the islands are found by sorting by start and tracking a running `MAX(end)` of previous rows; a row starts a new island when `start > running_max_end`.

### 6. Performance and design implications
- One sort: index `(partition_key, order_key)` (+ included columns) removes it.
- Dedupe/normalize *before* numbering; materialize `DISTINCT` days in a CTE or temp table for huge logs.
- Avoid the pre-window solution of self-joins with `NOT EXISTS` (O(n²) behavior); windows are typically orders of magnitude faster.
- Specify `ROWS UNBOUNDED PRECEDING` on running sums (faster than default `RANGE`, deterministic with ties).
- Use `DATEDIFF` on datetime2 carefully; `DATEDIFF(MINUTE, …)` counts boundary crossings, not elapsed time — use seconds if exactness matters.
- Persist islands (start, end, length) in a summary table if queried repeatedly.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ROW_NUMBER() OVER` | Dense counter | Delta trick |
| `LAG / LEAD` | Previous/next value | Flags, gaps, transitions |
| `SUM() OVER (… ROWS UNBOUNDED PRECEDING)` | Running total | Island ids from flags |
| `DATEADD(DAY, -rn, d)` | Constant per run | Date streaks |
| `DATEDIFF` | Difference between dates | Inactivity threshold |
| `GENERATE_SERIES` / calendar join | Complete expected series | Gap detection by anti-join |
| `MAX() OVER (… ROWS BETWEEN UNBOUNDED PRECEDING AND 1 PRECEDING)` | Running max of prior rows | Interval islands |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Delta trick (A/B): integer/date sequences, simple consecutiveness | Spacing is irregular (sessions) — use LAG flag |
| LAG + running sum: arbitrary thresholds, sessionization | Need pure SQL-92 portability |
| Two row numbers: runs of a repeating status | More than a few statuses — still works per status but verbose |
| Calendar anti-join for gaps | You only need adjacent gaps — `LEAD` is simpler |

### 9. Common mistakes
- No dedupe → duplicate dates break the constant → `SELECT DISTINCT` first.
- Treating "consecutive days" as "consecutive days above threshold" → streak includes breaks → flag and use two row numbers.
- Missing the first row (`LAG` is NULL) → first session not flagged → handle `IS NULL`.
- Ordering by a non-unique column → nondeterministic islands → add tie-breaker.
- `DATEDIFF(DAY…)` across time zones/midnight surprises → normalize to UTC dates.
- Using `COUNT(*)` for streak length on non-deduped rows → inflated.
- Running sum with default `RANGE` and ties → identical island ids across ties → `ROWS`.

### 10. 🎯 Interview kit
**30-second answer:** "I turn runs into groups by computing something constant per run — date minus row number, or a running sum of 'new run starts' flags from `LAG` — then `GROUP BY` that key and take `MIN`, `MAX` and `COUNT` as the island's start, end and length."

**Structure a longer answer:** clarify (what's consecutive, thresholds, duplicates, time zone) → mechanism (rn delta or LAG+sum) → trade-offs (index/sort, `ROWS` frames, vs self-join) → scenario (longest streak above threshold / sessions with 30-minute gap).

**Likely follow-ups:**
- *Why does `date − rn` work?* → both increase by 1 within a run.
- *Longest streak above a threshold vs consecutive days?* → need a flag; use two row numbers.
- *Find gaps?* → `LEAD` difference or calendar anti-join.
- *Alert flapping?* → `LAG(state)` transitions count.

**Signals interviewers look for:** dedupe step, framing awareness, thinking in sets instead of loops.

### 11. Check yourself
- [ ] Explain with a 5-row example why `date − rn` is constant in a streak.
- [ ] Write sessionization with a 30-minute timeout.
- [ ] Adapt the method for "above threshold" streaks.
- [ ] Find gaps in a numeric sequence with `LEAD`.
- [ ] Say what breaks without dedupe.
<!-- explainer:end -->

## Practice questions

- [Basic] [How would you compute the difference in days between two date columns?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40033_How_would_you_compute_the_difference_in_days_between_two_dat.md>)
- [Intermediate] [How would you use LAG to compute the change in a metric between consecutive readings per host (a simple delta)?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40978_How_would_you_use_LAG_to_compute_the_change_in_a_metric_betw.md>)
- [Intermediate] [How would you write SQL to measure a detector's lead time: for each true-positive flag, the minutes between the flag and the incident it predicted, and summarize the distribution to justify keeping the model?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-41080_How_would_you_write_SQL_to_measure_a_detector_s_lead_time_fo.md>)
- [Deep] [Write a query to calculate a running total of sales ordered by date.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40598_Write_a_query_to_calculate_a_running_total_of_sales_ordered_.md>)
- [Deep] [How would you write a query to roll up raw 10-second metrics into 1-minute averages for a rollup/downsampling table?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-41017_How_would_you_write_a_query_to_roll_up_raw_10-second_metrics.md>)
- [Deep] [Design a query to detect fraudulent behavior: users who made more than 5 failed logins from 3+ distinct IPs within a 10-minute window. Explain the windowing and why naive self-joins blow up.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40318_Design_a_query_to_detect_fraudulent_behavior_users_who_made_.md>)
- [Deep] [You want to suppress child alerts when a parent dependency is already firing (alert-storm noise reduction). How would you express this dependency-aware suppression in SQL using a self-join or NOT EXISTS against the topology?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-41060_You_want_to_suppress_child_alerts_when_a_parent_dependency_i.md>)
- [Deep] [You want to auto-generate a runbook-suggestion by finding the most common remediation that resolved past incidents with the same signature. How would you write that SQL and rank candidate actions by historical success rate?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-41121_You_want_to_auto-generate_a_runbook-suggestion_by_finding_th.md>)

## Related concepts

- [Time-based Windows](<../Window_Functions/Time-based_Windows.md>) *(Window Functions)*
- [UDF Performance](<../SQL_Functions/UDF_Performance.md>) *(SQL Functions)*
- [MySQL Optimizer Behavior](<../../../TOOLS/MySQL/MySQL_Platform/MySQL_Optimizer_Behavior.md>) *(MySQL Platform)*
- [Classic SQL Query Problems](<../../Database_Internals/DBMS_Concepts/Classic_SQL_Query_Problems.md>) *(DBMS Concepts)*
- [Distribution Change Alerts](<../../../02_Data_Engineering/Data_Quality/Data_Observability/Distribution_Change_Alerts.md>) *(Data Observability)*
- [Database Health Signals](<../../Database_Operations/Monitoring/Database_Health_Signals.md>) *(Monitoring)*

## References

- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — T-SQL for data analysis
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 3 Methods For Capturing Query Performance Metrics
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Service-Level Management › Be Careful Storing Averages!
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — E › F
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — T
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Automatic Updating
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page Compression › Performance Counters
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Forwarded Records per Second
- *Automating Data Quality Monitoring* — Jeremy Stanley, Paige Schwartz — Building a Model That Works on Real-World Data
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Exploring Databricks SQL › Setting Up Alerts
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Exploring Databricks SQL › Scheduling the Alert

