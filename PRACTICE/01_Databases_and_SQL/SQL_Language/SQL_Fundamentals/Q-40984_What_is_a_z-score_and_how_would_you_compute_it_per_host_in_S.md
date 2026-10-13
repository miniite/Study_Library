---
type: "practice"
difficulty: "Deep"
---

# What is a z-score, and how would you compute it per host in SQL to flag anomalous metric readings?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Data Completeness Metrics](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Data_Completeness_Metrics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The z-score is a statistical measurement that describes a value's relationship to the mean of a group of values. It tells us how many standard deviations an element is from the mean. In the context of AIOps, computing z-scores per host in SQL can help identify anomalous metric readings by comparing each reading against the historical average and deviation.

To compute z-scores per host in SQL, we need to follow these steps:

Calculate the mean (μ) for each metric across all hosts.
Calculate the standard deviation (σ) for each metric across all hosts.
For each metric reading of a specific host, calculate its z-score using the formula: z = (x - μ) / σ.

Let's break this down step by step from first principles:

Calculating the Mean and Standard Deviation: First, we need to compute the mean and standard deviation for each metric across all hosts. This involves aggregating the data for each metric and host combination.

Computing the Z-Score per Host: Once we have the mean and standard deviation, we can calculate the z-score for each metric reading of a specific host by subtracting the mean from the reading and dividing by the standard deviation.

Here’s how you would structure this in SQL:

SQL
-- Step 1: Create a CTE to hold aggregated data (mean and std dev)
WITH AggregatedData AS (
    SELECT
        metric_name,
        host_id,
        AVG(metric_value) OVER (PARTITION BY metric_name, host_id) AS mean,
        STDDEV(metric_value) OVER (PARTITION BY metric_name, host_id) AS std_dev
    FROM metrics_table
)

-- Step 2: Calculate z-score for each record in the dataset
SELECT
    md.host_id,
    md.metric_name,
    md.metric_value,
    (md.metric_value - ad.mean) / ad.std_dev AS z_score
FROM metrics_table md
JOIN AggregatedData ad ON md.metric_name = ad.metric_name AND md.host_id = ad.host_id;

By running this SQL query, we get a z-score for each metric reading per host. A high absolute value of the z-score indicates that the reading is far from the mean and could be flagged as an anomaly.

**Comparison:**

| host_id | metric_name | metric_value | z_score |
|---------|-------------|--------------|---------|
| 1       | CPU_util     | 70           | -1.2    |
| 1       | Memory_util   | 85           | 0.6     |
| 2       | Disk_space    | 90           | -2.3    |

**Code:**

```sql
WITH AggregatedData AS (
    SELECT
        metric_name,
        host_id,
        AVG(metric_value) OVER (PARTITION BY metric_name, host_id) AS mean,
        STDDEV(metric_value) OVER (PARTITION BY metric_name, host_id) AS std_dev
    FROM metrics_table
)

SELECT
    md.host_id,
    md.metric_name,
    md.metric_value,
    (md.metric_value - ad.mean) / ad.std_dev AS z_score
FROM metrics_table md
JOIN AggregatedData ad ON md.metric_name = ad.metric_name AND md.host_id = ad.host_id;
```

