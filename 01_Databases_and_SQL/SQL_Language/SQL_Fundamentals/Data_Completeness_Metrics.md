---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Data_Completeness_Metrics"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: []
---

# Data Completeness Metrics

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Completeness metrics are SQL measurements of "how much of what should be here is here" — null rates, missing-row (gap) counts and expected-vs-actual comparisons — computed per column, per partition and per time bucket so a regression is visible the day it starts.

### 1. The big picture
Data can be wrong in two ways: *values are bad* (validity, accuracy) or *values are missing* (completeness). Completeness is the cheapest quality dimension to measure and the one that most often breaks silently: a source stops sending one field, a partition never lands, a join drops rows. Because nothing errors, dashboards just show lower numbers.

Completeness checks come in three scales:
1. **Cell-level** — what fraction of a column is `NULL` (or empty string, or a sentinel like `'unknown'`, `-1`, `1900-01-01`)?
2. **Row-level** — did all the rows we expected arrive (expected row counts, sequence gaps, missing days or hosts)?
3. **Relationship-level** — does every fact have its dimension (orphans) and every entity have its expected children?

This lesson also covers the monitoring-flavoured relatives: per-host windows, and anomaly scoring against a host's own baseline.

### 2. Core concepts and vocabulary
- **Null rate** — `NULL` count ÷ row count for a column.
- **Completeness** — `1 − null_rate`, or `present / expected`.
- **Sentinel / disguised null** — a placeholder meaning "unknown" (`''`, `'N/A'`, `0`, `9999-12-31`) that bypasses `IS NULL` checks.
- **Expected set** — a reference list of what *should* exist (a calendar table, a host inventory, a list of source files).
- **Gap** — an expected key (day, host, sequence number) with no row.
- **Freshness** — how old the latest row is (`MAX(ts)` vs now); a completeness cousin.
- **Baseline** — the typical value for an entity (a host's 30-day mean/stddev), used for anomaly comparison.
- **Z-score** — `(x − mean) / stddev`; "3 standard deviations from the mean" is the simple threshold.

### 3. How it works, step by step
**Column null rate** — remember `COUNT(col)` skips NULLs while `COUNT(*)` doesn't:
1. `COUNT(*)` gives total rows.
2. `COUNT(col)` gives non-null rows.
3. `1.0 - COUNT(col) * 1.0 / NULLIF(COUNT(*),0)` is the null rate.

**Missing rows** — start from the *expected* set and `LEFT JOIN` the actuals; the unmatched expected rows are the gaps. You cannot find absence by scanning only the table that lacks the rows.

**Anomaly vs baseline** — compute mean and stddev over a trailing window per host, then compare the latest value; flag |z| > 3.

### 4. Worked example
```sql
-- 1) Null / sentinel rate per column, per day (one scan)
SELECT  CAST(load_ts AS date)                                           AS load_day,
        COUNT(*)                                                        AS row_cnt,
        1.0 * SUM(CASE WHEN email   IS NULL OR email   = '' THEN 1 ELSE 0 END) / COUNT(*) AS email_missing,
        1.0 * SUM(CASE WHEN country IS NULL OR country = 'N/A' THEN 1 ELSE 0 END) / COUNT(*) AS country_missing
FROM    dbo.customers_stage
GROUP BY CAST(load_ts AS date)
ORDER BY load_day;

-- 2) Missing metric samples: every host should report once per minute
WITH expected AS (
    SELECT h.host, m.minute_ts
    FROM   dbo.hosts h
    CROSS JOIN dbo.minute_calendar m           -- one row per minute in the window
    WHERE  m.minute_ts >= DATEADD(HOUR,-1,SYSUTCDATETIME())
)
SELECT e.host, COUNT(*) AS missing_minutes
FROM   expected e
LEFT JOIN dbo.metrics x
       ON x.host = e.host AND x.minute_ts = e.minute_ts AND x.metric_name = 'cpu'
WHERE  x.host IS NULL
GROUP BY e.host
HAVING COUNT(*) > 0
ORDER BY missing_minutes DESC;

-- 3) 3-sigma anomaly check against each host's own 30-day baseline
WITH base AS (
    SELECT host, AVG(value) AS mu, STDEV(value) AS sd
    FROM   dbo.metrics
    WHERE  metric_name = 'cpu'
      AND  ts >= DATEADD(DAY,-30,SYSUTCDATETIME()) AND ts < DATEADD(HOUR,-1,SYSUTCDATETIME())
    GROUP BY host
)
SELECT m.host, m.ts, m.value, (m.value - b.mu) / NULLIF(b.sd,0) AS z
FROM   dbo.metrics m
JOIN   base b ON b.host = m.host
WHERE  m.metric_name = 'cpu' AND m.ts >= DATEADD(HOUR,-1,SYSUTCDATETIME())
  AND  ABS((m.value - b.mu) / NULLIF(b.sd,0)) > 3;
```
Reading results: a rising `email_missing` on one `load_day` points to an upstream change that day; hosts with many `missing_minutes` have a dead agent or network partition; the baseline is computed on a window that *excludes* the hour being judged so an anomaly doesn't contaminate its own baseline.

### 5. Under the hood (optional at L4)
An index on `(host, ts)` (equality column first, range column second) makes per-host time-range reads a seek plus a contiguous range scan; it also provides the sorted order for per-host aggregates. `AVG` over a `NULL`-containing column silently divides by the non-null count, so a metric table with many NULL latencies will report a mean over only the *successful* rows — which is why "exclude NULL latencies" is both correct and something to disclose alongside the null rate. `STDEV` (sample) vs `STDEVP` (population) differ by a factor of √(n/(n−1)); for large n it is immaterial.

### 6. Performance and design implications
- One scan with `SUM(CASE …)` per column beats a query per column; with 50 columns generate the SQL from `INFORMATION_SCHEMA.COLUMNS`.
- Persist results to a small `dq_metrics(table, column, metric, day, value)` table and chart it; compute **incrementally** per new partition rather than re-profiling history.
- Use a **calendar/host inventory** as the expected set; it is the only reliable definition of "missing".
- Use a rolling window with `STDEV` carefully on big tables — pre-aggregate to 5-minute buckets first.
- Set an alert on *change* (today vs trailing 7-day median) rather than an absolute threshold, which breeds noise.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `COUNT(col)` vs `COUNT(*)` | Non-null vs all rows | Null rate |
| `NULLIF(x,'')` | Convert sentinel to NULL | Normalize before measuring |
| `LEFT JOIN … WHERE right.key IS NULL` | Anti-join | Missing expected rows |
| `AVG`, `STDEV`, `PERCENTILE_CONT` | Baselines | Anomaly thresholds |
| `INFORMATION_SCHEMA.COLUMNS` | Column list | Generate per-column checks |
| `sp_describe_first_result_set` | Result schema | Compare source vs target types |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Per-column null rate: cheap daily profile | Sentinels hide the real missingness — normalize first |
| Expected-set anti-join: calendar/inventory exists | There is no reliable inventory — fall back to "count dropped by >X% vs yesterday" |
| Z-score vs own baseline | Strong seasonality (weekday/hour) — compare to same hour last week |
| Absolute thresholds | Data volume changes — use ratios |

### 9. Common mistakes
- Measuring only `IS NULL` → disguised nulls pass → normalize with `NULLIF` or a rules table.
- Looking for missing rows in the table that lacks them → nothing found → anti-join from an expected set.
- Baseline includes the anomalous period → anomaly hidden → exclude the judged window.
- Averaging over NULLs without reporting the null rate → misleading mean → show both.
- `STDEV` of 0 → divide-by-zero → `NULLIF(sd,0)`.
- Profiling full history daily → expensive → incremental per partition.

### 10. 🎯 Interview kit
**30-second answer:** "I measure completeness at three levels: column null/sentinel rate with `COUNT(col)` vs `COUNT(*)`, missing rows by anti-joining an expected set (calendar × host list), and anomalies by comparing to each entity's own baseline. I store the metrics by day and alert on change."

**Structure a longer answer:** clarify (what is 'complete', SLA, what sentinels exist) → mechanism (aggregates, anti-join, z-score) → trade-offs (seasonality, cost, thresholds) → scenario (a host stops reporting; show detection query).

**Likely follow-ups:**
- *Why exclude NULL latencies from the average?* → `AVG` ignores them anyway, but a *zero-fill* would bias; report null rate separately.
- *Why index `(host, ts)`?* → equality then range → seek + contiguous scan.
- *3-sigma limits?* → assumes roughly normal data; use percentiles/MAD for skewed metrics.
- *How do you find days with no data at all?* → generate calendar, anti-join.

**Signals interviewers look for:** distinguishing NULL from sentinel, anti-join thinking, baselines per entity, avoiding contamination.

### 11. Check yourself
- [ ] Write a one-scan null-rate query for three columns.
- [ ] Find hosts that missed more than 5 of the last 60 minutes.
- [ ] Explain why the baseline window must exclude the judged window.
- [ ] Name two disguised nulls and how to normalize them.
- [ ] Decide absolute vs relative thresholds for a table whose volume doubles on weekends.
<!-- explainer:end -->

## Practice questions

- [Basic] [You have a metrics table (metric_name/host/value/ts). Write a query to fetch the last hour of CPU-usage rows for a single host.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40944_You_have_a_metrics_table_metric_name_host_value_ts_Write_a_q.md>)
- [Basic] [How do you join a hosts table with a metrics table to get each metric row alongside the host's data-center location?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40962_How_do_you_join_a_hosts_table_with_a_metrics_table_to_get_ea.md>)
- [Intermediate] [How do you compute the average API latency per endpoint from a request-log table, and why exclude NULL latencies?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40967_How_do_you_compute_the_average_API_latency_per_endpoint_from.md>)
- [Intermediate] [How would you write a query to detect metric values beyond 3 standard deviations from the mean as a simple anomaly check?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40983_How_would_you_write_a_query_to_detect_metric_values_beyond_3.md>)
- [Intermediate] [Why is an index on (host, ts) useful for a metrics table, and how does it speed up per-host time-range queries?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-41013_Why_is_an_index_on_host_ts_useful_for_a_metrics_table_and_ho.md>)
- [Deep] [What is a z-score, and how would you compute it per host in SQL to flag anomalous metric readings?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40984_What_is_a_z-score_and_how_would_you_compute_it_per_host_in_S.md>)
- [Deep] [How would you query an eval-results table to compute per-metric mean and confidence interval so you can tell a real regression from noise between two model versions?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40902_How_would_you_query_an_eval-results_table_to_compute_per-met.md>)
- [Deep] [Explain how you'd compute a per-host 'novelty' feature in SQL: how far the current metric vector is from the host's own 30-day typical range, so each host is scored against its own baseline not a global one.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-41111_Explain_how_you_d_compute_a_per-host_novelty_feature_in_SQL_.md>)

## Related concepts

- [Classic SQL Query Problems](<../../Database_Internals/DBMS_Concepts/Classic_SQL_Query_Problems.md>) *(DBMS Concepts)*
- [Time-based Windows](<../Window_Functions/Time-based_Windows.md>) *(Window Functions)*
- [Advanced Aggregation](<../Aggregation/Advanced_Aggregation.md>) *(Aggregation)*

## References

- *Azure Data Engineering Cookbook* — Ahmad Osama — Working with Relational Databases in Azure › Monitoring Azure SQL database metrics
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Operational Visibility › Instrumenting the Server or Instance
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Operational Visibility › Throughput and Latency Metrics
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — E
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — G › H

