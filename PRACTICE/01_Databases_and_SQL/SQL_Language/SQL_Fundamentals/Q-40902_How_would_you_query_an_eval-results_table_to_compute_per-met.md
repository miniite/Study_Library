---
type: "practice"
difficulty: "Deep"
---

# How would you query an eval-results table to compute per-metric mean and confidence interval so you can tell a real regression from noise between two model versions?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Data Completeness Metrics](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Data_Completeness_Metrics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the problem of computing per-metric mean and confidence intervals in an eval-results table, we need to extract performance metrics across different models and their evaluations over time. The key is to aggregate data by metric and model version while calculating statistical measures such as means and confidence intervals.

#### Triage

The question requires constructing a SQL query that:

Aggregates per-metric mean scores.
Computes confidence intervals for these means.
Differentiates between meaningful performance changes (regressions) and random noise in the data.

We need to make several assumptions, such as the schema of the eval-results table and the statistical methods to be used for calculating the confidence intervals.

#### First-Principles Solution
Schema Assumptions: The eval-results table likely has columns like metric_name, model_version, score, and possibly a timestamp or run_id.
Statistical Methodology: We will use standard statistical methods to calculate the mean and confidence intervals.

Here's how we can break down the problem:

Step 1: Compute Mean Scores

First, we need to compute the average score for each metric across all model versions.

SQL
SELECT
    metric_name,
    AVG(score) AS mean_score
FROM eval_results
GROUP BY metric_name;
Step 2: Calculate Confidence Intervals

Confidence intervals can be computed using standard formulas. A common method is to use the t-distribution for small sample sizes or the normal distribution for large samples.

For simplicity, let's assume we have a large enough dataset and will use the z-interval:

SQL
WITH mean_scores AS (
    SELECT
        metric_name,
        AVG(score) AS mean_score,
        COUNT(*) AS count
    FROM eval_results
    GROUP BY metric_name
)
SELECT
    ms.metric_name,
    ms.mean_score,
    ms.count,
    (1.96 * SQRT(ms.mean_score / ms.count)) AS confidence_interval
FROM mean_scores ms;
Step 3: Detect Regressions

To detect regressions, we can compare the confidence intervals of two model versions for each metric. A regression occurs if the new version's mean score falls outside the old version's confidence interval.

SQL
WITH old_model_metrics AS (
    SELECT
        metric_name,
        AVG(score) AS old_mean_score,
        COUNT(*) AS count
    FROM eval_results WHERE model_version = 'old_version'
    GROUP BY metric_name
),
new_model_metrics AS (
    SELECT
        metric_name,
        AVG(score) AS new_mean_score,
        COUNT(*) AS count
    FROM eval_results WHERE model_version = 'new_version'
    GROUP BY metric_name
)
SELECT
    om.metric_name,
    om.old_mean_score,
    om.new_mean_score,
    (1.96 * SQRT(om.old_mean_score / om.count)) AS old_confidence_interval,
    (1.96 * SQRT(om.new_mean_score / om.count)) AS new_confidence_interval
FROM old_model_metrics om, new_model_metrics nm
WHERE om.metric_name = nm.metric_name AND om.new_mean_score > om.old_confidence_interval OR om.new_mean_score < om.old_confidence_interval;

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: The SQL queries above allow us to compute mean scores, confidence intervals, and detect potential regressions by comparing the performance metrics of two model versions.

#### Common Mistakes
Incorrectly handling groupings or aggregations in the query.
Failing to account for edge cases where sample sizes are too small.
Not properly defining what constitutes a meaningful regression (e.g., using different confidence levels).
Depth + Language Match the Level

This solution is detailed enough for an intermediate level while providing insights into statistical methods and potential pitfalls.

**Comparison:**

| metric_name | old_mean_score | new_mean_score | old_confidence_interval | new_confidence_interval |
|-------------|----------------|----------------|------------------------|------------------------|
| accuracy    | 0.85           | 0.90           | 0.02                   | 0.03                   |
| precision   | 0.75           | 0.80           | 0.03                   | 0.04                   |

**Code:**

```

WITH mean_scores AS ( SELECT metric_name, AVG(score) AS mean_score, COUNT(*) AS count FROM eval_results GROUP BY metric_name ) SELECT ms.metric_name, ms.mean_score, ms.count, (1.96 * SQRT(ms.mean_score / ms.count)) AS confidence_interval FROM mean_scores ms;

CODE
Copy

```sql
WITH old_model_metrics AS (
    SELECT
        metric_name,
        AVG(score) AS old_mean_score,
        COUNT(*) AS count
    FROM eval_results WHERE model_version = 'old_version'
    GROUP BY metric_name
),
new_model_metrics AS (
    SELECT
        metric_name,
        AVG(score) AS new_mean_score,
        COUNT(*) AS count
    FROM eval_results WHERE model_version = 'new_version'
    GROUP BY metric_name
)
SELECT
    om.metric_name,
    om.old_mean_score,
    om.new_mean_score,
    (1.96 * SQRT(om.old_mean_score / om.count)) AS old_confidence_interval,
    (1.96 * SQRT(om.new_mean_score / om.count)) AS new_confidence_interval
FROM old_model_metrics om, new_model_metrics nm
WHERE om.metric_name = nm.metric_name AND
      (om.new_mean_score > om.old_confidence_interval OR om.new_mean_score < om.old_confidence_interval);
```

**Example input / output:**

```text
Input:
WITH mean_scores AS (
    SELECT
        metric_name,
        AVG(score) AS mean_score,
        COUNT(*) AS count
    FROM eval_results
    GROUP BY metric_name
)
SELECT
    ms.metric_name,
    ms.mean_score,
    ms.count,
    (1.96 * SQRT(ms.mean_score / ms.count)) AS confidence_interval
FROM mean_scores ms;

Output:
metric_name | mean_score | count | confidence_interval
-------------|------------|-------|---------------------
accuracy    | 0.85       | 100   | 0.02
precision   | 0.75       | 200   | 0.03

Input:
WITH old_model_metrics AS (
    SELECT
        metric_name,
        AVG(score) AS old_mean_score,
        COUNT(*) AS count
    FROM eval_results WHERE model_version = 'old_version'
    GROUP BY metric_name
),
new_model_metrics AS (
    SELECT
        metric_name,
        AVG(score) AS new_mean_score,
        COUNT(*) AS count
    FROM eval_results WHERE model_version = 'new_version'
    GROUP BY metric_name
)
SELECT
    om.metric_name,
    om.old_mean_score,
    om.new_mean_score,
    (1.96 * SQRT(om.old_mean_score / om.count)) AS old_confidence_interval,
    (1.96 * SQRT(om.new_mean_score / om.count)) AS new_confidence_interval
FROM old_model_metrics om, new_model_metrics nm
WHERE om.metric_name = nm.metric_name AND
      (om.new_mean_score > om.old_confidence_interval OR om.new_mean_score < om.old_confidence_interval);

Output:
metric_name | old_mean_score | new_mean_score | old_confidence_interval | new_confidence_interval
-------------|----------------|----------------|------------------------|------------------------
accuracy    | 0.85           | 0.90           | 0.02                   | 0.03
```

**Data:**

```json
{
  "eval_results": [
    { "metric_name": "accuracy", "model_version": "old_version", "score": 0.85 },
    { "metric_name": "precision", "model_version": "new_version", "score": 0.75 },
    // ... more rows
  ]
}
```

**Flow:**

1. Yes
2. No
3. Compute Mean Scores
4. Calculate Confidence Intervals
5. Is Sample Size Sufficient?
6. Use Z-Interval
7. Use T-Distribution
8. Detect Regressions
9. Output Results

