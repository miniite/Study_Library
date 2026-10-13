---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Retention_Queries"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: []
---

# Retention Queries

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Retention queries answer "of the users who did X in period A, how many came back in period B?" by assigning each user to a cohort (their first period), then counting distinct returners per later period, always on a dense grid so zeros show as zeros.

### 1. The big picture
Product and growth teams live on retention numbers: daily/weekly active users (DAU/WAU), "7-day retention", churn flags, tenure, recency. They are all variations on three SQL moves:

1. **Windowing by time**: restrict rows to "the last N days" or bucket them into days/weeks/months.
2. **Per-user first/last facts**: first event (cohort), last event (recency/churn), count of distinct active days (engagement).
3. **Joining a user's later activity back to the cohort** and counting distinct users, then dividing by cohort size.

The mistakes are mostly about *time arithmetic* (off-by-one, time zones, non-sargable predicates), *grain* (counting events instead of distinct users), and *missing rows* (a week with zero returners simply does not appear unless you build the grid).

### 2. Core concepts and vocabulary
- **Event table** — one row per user action: `(user_id, event_ts, ...)`.
- **Active user** — a user with at least one qualifying event in the period.
- **DAU / WAU / MAU** — distinct active users per day / week / month.
- **Cohort** — a group of users who share a starting period (e.g. signup week).
- **Cohort size** — number of users in the cohort (the denominator).
- **Period index (age)** — periods elapsed since the cohort period: 0, 1, 2, …
- **Retention rate** — returners in period *k* ÷ cohort size.
- **Churned** — no activity in the last N days (a definition you must state).
- **Recency / tenure** — days since last event / days between first and last event.
- **Half-open interval** — `ts >= start AND ts < end`; the safe way to express a time window.
- **Sessionization** — grouping a user's events into sessions separated by an inactivity gap.

### 3. How it works, step by step
**A. Windows.** "Last 90 days" is `event_ts >= CURRENT_DATE - INTERVAL '90 days'` (PostgreSQL) or `event_ts >= DATEADD(day, -90, CAST(GETDATE() AS date))` (SQL Server). Always compare the *bare column* against an expression of constants so an index on `event_ts` can be used. `WHERE DATEDIFF(day, event_ts, GETDATE()) <= 90` forces a computation per row and cannot seek.

**B. Bucketing.** Truncate timestamps to the period: `date_trunc('week', event_ts)` (PostgreSQL), `DATE_TRUNC('week', ...)` (Snowflake/BigQuery use similar), `DATEADD(week, DATEDIFF(week, 0, event_ts), 0)` (SQL Server). Note week start differs by engine/settings (Monday vs Sunday).

**C. Cohort retention (the standard triangle).**
1. Compute each user's first period (`MIN` of bucketed event, or signup date).
2. Compute each user's distinct active periods (distinct `(user_id, period)`).
3. Join (2) to (1) and compute `age = period - cohort_period`.
4. `COUNT(DISTINCT user_id)` per `(cohort, age)`.
5. Divide by cohort size; pivot or leave long for the heatmap.
6. To show zeros, cross join cohorts with an `age` generator and left join counts.

**D. Churn label and recency.** `MAX(event_ts)` per user, compared with the reference date. State the reference date explicitly (`as_of`) rather than using `NOW()` so training data is reproducible.

**E. Sessionization.** Order each user's events, compute the gap from the previous event with `LAG`, flag a new session when the gap exceeds 30 minutes, then take a running `SUM` of the flags to number sessions.

### 4. Worked example
Schema: `users(user_id, signup_date)` and `events(user_id, event_ts)`.

```sql
-- Weekly cohort retention triangle (PostgreSQL)
WITH cohort AS (
  SELECT user_id,
         date_trunc('week', signup_date)::date AS cohort_week
  FROM   users
),
activity AS (
  SELECT DISTINCT user_id,
         date_trunc('week', event_ts)::date AS active_week
  FROM   events
),
sizes AS (
  SELECT cohort_week, COUNT(*) AS cohort_size
  FROM   cohort
  GROUP  BY cohort_week
),
ret AS (
  SELECT c.cohort_week,
         (a.active_week - c.cohort_week) / 7 AS week_age,
         COUNT(DISTINCT a.user_id)          AS returners
  FROM   cohort c
  JOIN   activity a
         ON a.user_id = c.user_id
        AND a.active_week >= c.cohort_week
  GROUP  BY c.cohort_week, week_age
),
grid AS (                                   -- dense grid so zeros appear
  SELECT s.cohort_week, s.cohort_size, g AS week_age
  FROM   sizes s
  CROSS  JOIN generate_series(0, 12) AS g
  WHERE  s.cohort_week + g * 7 <= date_trunc('week', current_date)
)
SELECT grid.cohort_week,
       grid.week_age,
       COALESCE(ret.returners, 0)                              AS returners,
       ROUND(100.0 * COALESCE(ret.returners, 0) / grid.cohort_size, 1) AS retention_pct
FROM   grid
LEFT   JOIN ret USING (cohort_week, week_age)
ORDER  BY grid.cohort_week, grid.week_age;
```

How to read it: `week_age = 0` is the signup week and is normally 100% if signup counts as activity. Each row of a cohort falls as age increases. The last `WHERE` keeps the triangle shape by excluding periods that have not happened yet (otherwise recent cohorts show misleading zeros).

Per-user features in one pass:

```sql
-- as_of fixed so the result is reproducible
SELECT user_id,
       MIN(event_ts)                                              AS first_event,
       MAX(event_ts)                                              AS last_event,
       COUNT(DISTINCT event_ts::date) FILTER (WHERE event_ts >= DATE '2026-09-01' - 30
                                                AND event_ts <  DATE '2026-09-01') AS active_days_30d,
       DATE '2026-09-01' - MAX(event_ts)::date                    AS days_since_last,
       CASE WHEN MAX(event_ts) < DATE '2026-09-01' - 30 THEN 1 ELSE 0 END AS churned
FROM   events
WHERE  event_ts < DATE '2026-09-01'           -- never read past the as-of date (no leakage)
GROUP  BY user_id;
```

### 5. Under the hood
- `COUNT(DISTINCT ...)` is usually implemented by sorting or hashing the pairs; on a billion-row table it dominates cost. Deduplicate once in a CTE (`SELECT DISTINCT user_id, period`) so later aggregates are plain `COUNT(*)`, or use an approximate distinct (HyperLogLog) for dashboards.
- The date predicate on a clustered/B-tree index becomes a range seek only when the column is untouched. Wrapping it in `CAST(... AS date)` is often sargable in SQL Server (it has a special rule for date casts) but a generic function is not.
- Partition tables by event date: a "last 7 days" query then reads 7 partitions, and the retention join can prune.
- Incremental approach: maintain a daily `user_activity_days(user_id, activity_date)` table (one row per user per day). Every retention query then reads that small table instead of raw events.

### 6. Performance and design implications
- Aggregate to `(user, day)` first; almost all retention metrics derive from it.
- Use half-open ranges; never `BETWEEN` on timestamps (`BETWEEN '2026-01-01' AND '2026-01-31'` drops Jan 31 events after midnight).
- Decide the time zone before bucketing (see the timestamp lesson): UTC day vs the user's local day changes DAU by a few percent.
- Cohort size must come from the *cohort* table, not from the activity join, otherwise users who never returned vanish from the denominator.
- Check with `EXPLAIN`/execution plan that the event-time filter turns into a seek or partition prune.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `date_trunc('week', ts)` / `DATETRUNC` (SQL Server 2022) | Bucket a timestamp | Cohorts, WAU |
| `DATEADD` / `DATEDIFF` | Date arithmetic (SQL Server) | Windows, ages |
| `COUNT(DISTINCT x)` / `COUNT(DISTINCT x) FILTER (WHERE …)` | Distinct users | DAU, returners |
| `LAG(ts) OVER (PARTITION BY user ORDER BY ts)` | Previous event | Gaps, sessionization |
| `generate_series` / numbers table / `GENERATE_SERIES` (SQL Server 2022) | Build a dense grid | Zero-filled triangles |
| `MIN`/`MAX` per user | First / last event | Tenure, recency, churn |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Compute from raw events | Events table is huge and queries repeat; pre-aggregate to user-day |
| Bounded retention ("active exactly in week k") | You need "retained at least once on/after" (unbounded) — different metric, state which |
| Cohort by signup date | Signup is not the "start" (use first meaningful action) |
| `NOW()` in the query | You need reproducible training sets — use a fixed `as_of` |
| Approximate distinct | Finance/regulatory reporting that must be exact |

### 9. Common mistakes
- Counting rows instead of distinct users → retention > 100% → `COUNT(DISTINCT user_id)` after deduping.
- Inner-joining activity to cohorts → weeks with no returners disappear → build the grid and `LEFT JOIN`.
- `BETWEEN` on timestamps → last day truncated → half-open interval.
- Using local `NOW()` for churn → numbers change on every re-run → pass an explicit as-of date.
- Reading events after the prediction date when building features → leakage → filter `event_ts < as_of`.
- Mixing time zones in bucketing → day boundaries differ per source → normalize to one zone first.

### 10. 🎯 Interview kit
**30-second answer:** Assign each user a cohort (first period), reduce activity to distinct `(user, period)`, join on user, compute age = period − cohort, count distinct users per `(cohort, age)`, divide by cohort size, and use a dense grid so empty periods show 0.

**Structure a longer answer:** clarify definition (active, retained, period, time zone) → grain and CTE plan → the SQL → performance (user-day table, partitions, sargable filters) → edge cases (zeros, incomplete periods, leakage).

**Likely follow-ups:**
- *Why not `BETWEEN`?* → inclusive upper bound plus time portion.
- *How do you sessionize?* → `LAG`, gap flag, running sum.
- *How to speed it up at 10 B events?* → daily user-activity table, partitions, approximate distinct.
- *Rolling vs classic retention?* → "active on day N" vs "active on or after day N".

**Signals interviewers look for:** explicit definitions, correct denominator, zero-filling, sargable windows, awareness of time zones and leakage.

### 11. Check yourself
- [ ] Write a 7-day rolling DAU average using a daily distinct-users CTE.
- [ ] Explain why an inner join makes the triangle wrong.
- [ ] Rewrite `WHERE DATEDIFF(day, ts, GETDATE()) < 30` so it can use an index.
- [ ] Build a churn flag as of a fixed date without data leakage.
- [ ] Sessionize an event stream with a 30-minute gap.
<!-- explainer:end -->

## Practice questions

- [Basic] [How would you write a query to find all employees hired in the last 90 days relative to today?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39865_How_would_you_write_a_query_to_find_all_employees_hired_in_t.md>)
- [Basic] [How would you pull a training dataset of the last 90 days of user transactions from a SQL table?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40500_How_would_you_pull_a_training_dataset_of_the_last_90_days_of.md>)
- [Basic] [How would you compute the number of distinct days a user was active in the last 30 days as an engagement feature?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40589_How_would_you_compute_the_number_of_distinct_days_a_user_was.md>)
- [Intermediate] [What SQL would you write to create a binary label column: 1 if a user churned (no activity in 30 days), else 0?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40510_What_SQL_would_you_write_to_create_a_binary_label_column_1_i.md>)
- [Intermediate] [How would you write a query to compute average session duration per user, handling sessions with a NULL end time?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40596_How_would_you_write_a_query_to_compute_average_session_durat.md>)
- [Deep] [Explain how you'd compute cohort retention (signup week vs weeks-since-signup) entirely in SQL and shape the result so a frontend renders a triangle heatmap directly.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39953_Explain_how_you_d_compute_cohort_retention_signup_week_vs_we.md>)
- [Deep] [How would you sessionize a raw event stream in SQL, starting a new session after 30 minutes of inactivity per user?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40064_How_would_you_sessionize_a_raw_event_stream_in_SQL_starting_.md>)
- [Deep] [How would you write a query to compute a weekly cohort of new signups for a retention analysis feeding a retention model?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40602_How_would_you_write_a_query_to_compute_a_weekly_cohort_of_ne.md>)

## Related concepts

- [Time-based Windows](<../Window_Functions/Time-based_Windows.md>) *(Window Functions)*
- [Classic SQL Query Problems](<../../Database_Internals/DBMS_Concepts/Classic_SQL_Query_Problems.md>) *(DBMS Concepts)*

## References

- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Storage Design Patterns in Data Engineering › 5. Storage Design Patterns in Data Engineering
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — SQL Graph (cont.)
- *Data Engineering Design Patterns* — Bartosz Konieczny — Chapter 5. Data Value Design Patterns › Problem
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Solving Queries Using Accumulating Aggregates › The Subscription Problem
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — integer, but it is typically much more useful to input a timestamp using one of several supported

