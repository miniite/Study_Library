---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd compute a per-host 'novelty' feature in SQL: how far the current metric vector is from the host's own 30-day typical range, so each host is scored against its own baseline not a global one.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Data Completeness Metrics](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Data_Completeness_Metrics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute a per-host 'novelty' feature in SQL, we need to measure how far the current metric vector of each host deviates from its typical 30-day range. This involves calculating the mean and standard deviation of historical data for each host over a 30-day window. The novelty score can then be calculated as the Euclidean distance between the current vector and the historical baseline.

The key steps are:

Calculate Host Baselines: Compute the mean and standard deviation of metric values for each host.
Compute Novelty Score: For the current day, calculate how far the metric vector deviates from these baselines using a distance metric like Euclidean distance.

Let's break this down in more detail:

**Code:**

```sql
-- Step 1: Calculate Host Baselines

WITH host_metrics AS (
    SELECT
        host_id,
        metric_value,
        DATE_TRUNC('day', timestamp) AS day
    FROM
        metrics_table
),
host_daily_metrics AS (
    SELECT
        host_id,
        day,
        metric_value,
        AVG(metric_value) OVER (PARTITION BY host_id, day ORDER BY timestamp ROWS BETWEEN 29 PRECEDING AND CURRENT ROW) AS mean_30_day_metric,
        STDDEV_SAMP(metric_value) OVER (PARTITION BY host_id, day ORDER BY timestamp ROWS BETWEEN 29 PRECEDING AND CURRENT ROW) AS std_30_day_metric
    FROM
        host_metrics
)
```

**Example input / output:**

```text
-- Step 2: Compute Novelty Score for Current Day

SELECT
    m.host_id,
    m.timestamp,
    (m.metric_value - h.mean_30_day_metric) / h.std_30_day_metric AS novelty_score
FROM
    metrics_table m
JOIN
    host_daily_metrics h ON m.host_id = h.host_id AND DATE_TRUNC('day', m.timestamp) = h.day;
```

**Flow:**

1. Yes
2. No
3. Compute Host Baselines
4. Are we on the current day?
5. Calculate Novelty Score
6. Wait for next 30-day windowto pass
7. Output novelty scores for allhosts

