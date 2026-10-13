---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a query to detect metric values beyond 3 standard deviations from the mean as a simple anomaly check?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Data Completeness Metrics](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Data_Completeness_Metrics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To detect metric values that are more than 3 standard deviations away from the mean, we need to calculate both the mean and the standard deviation of the metrics first. Then, we can query the database to filter out the values that fall outside these bounds. This approach is a simple yet effective way to identify anomalies in time series data.

#### Triaging the Question

The question asks us to write an SQL query for detecting anomalies. Anomalies here are defined as metric values exceeding 3 standard deviations from the mean. The interviewer is probing your understanding of basic statistical concepts, ability to write efficient SQL queries, and handling of database operations.

#### Assumptions
and First-Principles Solution
Data Storage: Assume we have a table metrics with at least two columns: timestamp (to track the time when each metric was recorded) and value (the actual value of the metric).
Statistics Calculation: We need to calculate the mean (μ) and standard deviation (σ) of the metrics.
Anomaly Detection: Values more than 3 standard deviations from the mean are flagged as anomalies.
Step-by-Step Solution

Calculate Mean and Standard Deviation:

Calculate the mean μ of all values in the value column.
Calculate the standard deviation σ using a SQL function.

Filter Anomalies: Use these statistical measures to filter out metric values that lie outside 3 standard deviations from the mean.

SQL Query Implementation
SQL
-- Step 1: Calculate Mean and Standard Deviation
WITH stats AS (
    SELECT
        AVG(value) OVER () AS mean,
        STDDEV(value) OVER () AS std_dev
    FROM metrics
)
-- Step 2: Filter Anomalies
SELECT
    timestamp, value
FROM metrics
JOIN stats ON 1=1
WHERE value > (mean + 3 * std_dev) OR value < (mean - 3 * std_dev);

#### Closing Summary
and Common Mistakes/Follow-Ups

This query effectively identifies anomalies by filtering out metric values that are more than 3 standard deviations from the mean. However, there are a few common pitfalls to be aware of:

Data Quality: Ensure your data is clean and free of outliers before applying this method.

#### Performance Considerations
For very large datasets, window functions (like OVER ()) can be resource-intensive. Optimizing queries or using sampling techniques might be necessary.
Contextual Adjustments: In practice, thresholds like 3 standard deviations may need to be adjusted based on specific business requirements.

#### Common Mistakes
Not accounting for data quality issues.
Misunderstanding the statistical concepts (e.g., confusing mean and median).
Failing to optimize queries for large datasets.
Further Discussion Points
How would you handle non-normal distributions?
What if the dataset is very large, how would you optimize this query?
Are there better methods or thresholds to use in certain scenarios?

**Code:**

```sql
WITH stats AS (
    SELECT
        AVG(value) OVER () AS mean,
        STDDEV(value) OVER () AS std_dev
    FROM metrics
)
SELECT
    timestamp, value
FROM metrics
JOIN stats ON 1=1
WHERE value > (mean + 3 * std_dev) OR value < (mean - 3 * std_dev);
```

