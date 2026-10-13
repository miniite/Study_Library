---
type: "practice"
difficulty: "Deep"
---

# What are the LAG and LEAD window functions and how would you compute a day-over-day change with them?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Date Arithmetic](<../../../../TOOLS/MySQL/MySQL_Platform/Date_Arithmetic.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle the problem of computing a day-over-day change using MySQL's LAG and LEAD window functions, let’s break down the key concepts and walk through an example from first principles.

TL;DR:

The LAG function allows you to access values from the previous row within a partition, while the LEAD function does the same for the next row. By leveraging these functions, we can compute a day-over-day change, which represents how a metric has changed from one day to another.

#### Triaging the Question

The question is asking about using MySQL's window functions, specifically LAG, to calculate day-over-day changes. The core of the problem involves understanding how to access and compare data across rows using window functions, and then applying this to compute differences.

#### Assumptions
We have a table with daily metrics.
Each row in the table represents a single day’s metric value.
Step-by-Step Solution:

Understanding LAG:

The LAG function is used to access the previous row's data within the same partition.
Syntax: LAG(column_name, offset, default_value) OVER (PARTITION BY some_column)

Computing Day-over-Day Change:

To compute a day-over-day change, we need to subtract the current row’s value from the previous row’s value using LAG.
The formula would be: current_value - LAG(current_value)

#### Example
SQL Query:
SQL
SELECT
    date,
    metric_value AS today_metric,
    LAG(metric_value) OVER (ORDER BY date) AS prev_metric,
    (metric_value - LAG(metric_value) OVER (ORDER BY date)) AS day_over_day_change
FROM
    daily_metrics;
Explanation:
date: The date for the current row.
metric_value AS today_metric: The metric value on the given date.
LAG(metric_value) OVER (ORDER BY date) AS prev_metric: Accesses the previous day’s metric value based on the date ordering.
(metric_value - LAG(metric_value) OVER (ORDER BY date)) AS day_over_day_change: Computes the difference between today's and yesterday's metric values.

#### Closing Summary

By using LAG, we can easily compute day-over-day changes in metrics stored daily. This approach ensures that we correctly account for each value’s predecessor, providing a clear measure of change over time.

#### Common Mistakes/Follow-Ups
Mistake: Forgetting to order the rows properly with ORDER BY. Without proper ordering, LAG will return incorrect values.

#### Follow-Up
Exploring different partitioning strategies if the data spans multiple segments or has irregularities.
Advanced Follow-Up: Considering performance optimizations for large datasets using index scans and other database tuning techniques.

**Comparison:**

```
table daily_metrics {
    date | metric_value
    2023-10-01 | 100
    2023-10-02 | 150
    2023-10-03 | 200
}
```

**Code:**

```sql
SELECT
    date,
    metric_value AS today_metric,
    LAG(metric_value) OVER (ORDER BY date) AS prev_metric,
    (metric_value - LAG(metric_value) OVER (ORDER BY date)) AS day_over_day_change
FROM
    daily_metrics;
```

**Example input / output:**

```text
Input:
Date       Metric Value   Previous Day's Metric   Day-over-Day Change
2023-10-01 100            NULL                    NULL
2023-10-02 150            100                     +50
2023-10-03 200            150                     +50
```

**Data:**

```json
{
  "daily_metrics": [
    {"date": "2023-10-01", "metric_value": 100},
    {"date": "2023-10-02", "metric_value": 150},
    {"date": "2023-10-03", "metric_value": 200}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Date
4. Is this the first row?
5. Day-over-Day Change = NULL
6. Metric Value - LAG(MetricValue)
7. Day Over Day Change

