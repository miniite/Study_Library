---
type: "practice"
difficulty: "Expert"
---

# Explain how you'd design SQL to compute features for a time-series forecasting model (lags, rolling means, calendar features) while strictly respecting the forecast horizon to avoid look-ahead.

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Analytical SQL for ML Features](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Analytical_SQL_for_ML_Features.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design SQL for computing features in a time-series forecasting model while respecting the forecast horizon and avoiding look-ahead, we need to ensure that our feature calculations are based only on historical data up to the point of the prediction. Here's a step-by-step approach:

#### Overview
We'll use window functions in SQL to compute lagged values and rolling statistics from historical data.

#### Triage
The main challenge is to define a method for calculating features such as lags, rolling means, and calendar-based features without looking into the future.

#### First-Principles Solution
Lag Features: We use LAG() window function to create lagged values.
Rolling Means: Use AVG() OVER (ORDER BY ... ROWS BETWEEN ...).
Calendar Features: Extract date components like day of the week, month, etc., using SQL date functions.

Let's dive into each step with concrete examples and explanations.

Step-by-Step Solution
1. Lag Features
SQL
SELECT
    ts,
    value,
    LAG(value, n) OVER (ORDER BY ts) AS lag_value_n
FROM
    time_series_data;

Here, n is the number of periods back we want to look for each feature.

2. Rolling Means
SQL
WITH rolling_stats AS (
    SELECT
        ts,
        value,
        AVG(value) OVER (ORDER BY ts ROWS BETWEEN n PRECEDING AND CURRENT ROW) AS rolling_mean_n
    FROM
        time_series_data
)
SELECT * FROM rolling_stats;

Here, n is the number of periods over which to compute the mean.

3. Calendar Features
SQL
SELECT
    ts,
    value,
    EXTRACT(DAYOFWEEK FROM ts) AS day_of_week,
    TO_CHAR(ts, 'MM') AS month,
    TO_CHAR(ts, 'YYYY-MM-DD') AS date
FROM
    time_series_data;

These calendar features can be used to capture temporal patterns.

Combining Everything
SQL
WITH lag_and_rolling_stats AS (
    SELECT
        ts,
        value,
        LAG(value, 1) OVER (ORDER BY ts) AS lag_value_1,
        AVG(value) OVER (ORDER BY ts ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS rolling_mean_2
    FROM
        time_series_data
)
SELECT
    ts,
    value,
    lag_value_1,
    rolling_mean_2,
    EXTRACT(DAYOFWEEK FROM ts) AS day_of_week,
    TO_CHAR(ts, 'MM') AS month,
    TO_CHAR(ts, 'YYYY-MM-DD') AS date
FROM
    lag_and_rolling_stats;

#### Closing Summary

By using window functions and carefully defining the frame for each calculation, we can ensure that all features are computed based on historical data. This approach avoids look-ahead bias, ensuring our model is trained on unbiased data.

#### Common Mistakes/Follow-Ups
Incorrect Frame Definition: Not properly setting the ROWS BETWEEN clause can lead to incorrect rolling statistics.
Lag Calculation Mismatch: Using an incorrect lag value (n) will misalign features with the actual time series.
Calendar Feature Extraction Issues: Failing to correctly extract date components or using them incorrectly in feature engineering.

**Comparison:**

| ts       | value | day_of_week | month     | date           |
|----------|-------|-------------|-----------|----------------|
| 2023-01-01 | 10    | 6           | Jan       | 2023-01-01     |
| 2023-01-02 | 15    | 7           | Jan       | 2023-01-02     |
| 2023-01-03 | 20    | 1           | Jan       | 2023-01-03     |
| 2023-01-04 | 25    | 2           | Jan       | 2023-01-04     |

**Code:**

```sql
WITH lag_and_rolling_stats AS (
    SELECT
        ts,
        value,
        LAG(value, 1) OVER (ORDER BY ts) AS lag_value_1,
        AVG(value) OVER (ORDER BY ts ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS rolling_mean_2
    FROM
        time_series_data
)
SELECT
    ts,
    value,
    lag_value_1,
    rolling_mean_2,
    EXTRACT(DAYOFWEEK FROM ts) AS day_of_week,
    TO_CHAR(ts, 'MM') AS month,
    TO_CHAR(ts, 'YYYY-MM-DD') AS date
FROM
    lag_and_rolling_stats;
```

**Example input / output:**

```text
Input:
+------------+-------+
| ts         | value |
+------------+-------+
| 2023-01-01 | 10    |
| 2023-01-02 | 15    |
| 2023-01-03 | 20    |
| 2023-01-04 | 25    |
+------------+-------+

Output:
+------------+-------+----------+--------------+
| ts         | value | lag_value_1 | rolling_mean_2 |
+------------+-------+----------+--------------+
| 2023-01-01 | 10    | NULL      | NULL          |
| 2023-01-02 | 15    | 10        | 11.67         |
| 2023-01-03 | 20    | 15        | 14            |
| 2023-01-04 | 25    | 20        | 18            |
+------------+-------+----------+--------------+

Calendar Features:
+------------+-------+----------+-----------------+----+----+-------------------+
| ts         | value | lag_value_1 | rolling_mean_2 | day_of_week | month | date              |
+------------+-------+----------+--------------+----------+--------+-------------------+
| 2023-01-01 | 10    | NULL      | NULL          | 6         | Jan    | 2023-01-01        |
| 2023-01-02 | 15    | 10        | 11.67         | 7         | Jan    | 2023-01-02        |
| 2023-01-03 | 20    | 15        | 14            | 1         | Jan    | 2023-01-03        |
| 2023-01-04 | 25    | 20        | 18            | 2         | Jan    | 2023-01-04        |
+------------+-------+----------+--------------+----------+--------+-------------------+
```

**Data:**

```json
[
    {"ts": "2023-01-01", "value": 10},
    {"ts": "2023-01-02", "value": 15},
    {"ts": "2023-01-03", "value": 20},
    {"ts": "2023-01-04", "value": 25}
]
```

**Flow:**

1. Input Time Series Data
2. Lag Features
3. LAG(value, n)
4. Rolling Means
5. AVG(value) OVER (ORDER BY tsROWS BETWEEN n PRECEDINGAND CURRENT ROW)
6. Calendar Features
7. EXTRACT(DAYOFWEEK FROMts), TO_CHAR(ts, 'MM'),TO_CHAR(ts, 'YYYY-MM-DD')
8. Combined Output

