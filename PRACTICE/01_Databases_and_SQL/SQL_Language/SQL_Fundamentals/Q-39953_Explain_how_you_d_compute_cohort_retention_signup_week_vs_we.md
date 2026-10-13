---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd compute cohort retention (signup week vs weeks-since-signup) entirely in SQL and shape the result so a frontend renders a triangle heatmap directly.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Retention Queries](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Retention_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To calculate cohort retention using SQL, we first need to understand how cohorts are formed and tracked over time. A cohort is defined by a shared characteristic at a specific point in time; in this case, it's the week of signup. The goal is to compute the retention rate for each cohort as weeks pass, shaping the result into a triangle heatmap that can be rendered directly on a frontend.

#### Triaging the Question

The question requires us to:

Define Cohorts: Group users by their signup week.
Track Retention: Measure how many users remain active in subsequent weeks relative to their signup week.
Shape Data for Visualization: Format the data so it can be directly used to create a triangle heatmap.

#### First-Principles Solution

We'll use SQL to:

Create cohort buckets based on signup week.
Count active users per cohort and week.
Compute retention rates for each user in their respective cohort over weeks.
Aggregate the data into a format suitable for heatmap visualization.

Let's assume we have a table user_activity with columns: user_id, signup_week, activity_week, and is_active. We'll start by creating cohort buckets using a common range or specific weeks.

Step-by-Step Solution
1. Create Cohort Buckets

We can create cohorts based on the signup week:

SQL
WITH cohorts AS (
    SELECT
        user_id,
        signup_week,
        activity_week,
        is_active
    FROM
        user_activity
),
cohort_buckets AS (
    SELECT
        signup_week AS cohort_week,
        activity_week - signup_week + 1 AS retention_period
    FROM
        cohorts
)
2. Count Active Users Per Cohort and Week

Next, we count active users in each cohort:

SQL
WITH cohorts AS (
    SELECT
        user_id,
        signup_week,
        activity_week,
        is_active
    FROM
        user_activity
),
cohort_buckets AS (
    SELECT
        signup_week AS cohort_week,
        activity_week - signup_week + 1 AS retention_period
    FROM
        cohorts
),
user_counts AS (
    SELECT
        cohort_week,
        retention_period,
        COUNT(*) FILTER (WHERE is_active) AS active_users
    FROM
        cohorts
    GROUP BY
        cohort_week, retention_period
)
3. Compute Retention Rates

We then compute the percentage of users who remain active:

SQL
WITH cohorts AS (
    SELECT
        user_id,
        signup_week,
        activity_week,
        is_active
    FROM
        user_activity
),
cohort_buckets AS (
    SELECT
        signup_week AS cohort_week,
        activity_week - signup_week + 1 AS retention_period
    FROM
        cohorts
),
user_counts AS (
    SELECT
        cohort_week,
        retention_period,
        COUNT(*) FILTER (WHERE is_active) AS active_users
    FROM
        cohorts
    GROUP BY
        cohort_week, retention_period
),
retention_rates AS (
    SELECT
        cohort_week,
        retention_period,
        COUNT(DISTINCT user_id) OVER (PARTITION BY cohort_week) AS total_users,
        active_users::float / COUNT(DISTINCT user_id) OVER (PARTITION BY cohort_week) * 100 AS retention_rate
    FROM
        user_counts
)
4. Shape Data for Heatmap Visualization

Finally, we reshape the data to fit a triangle heatmap format:

SQL
WITH cohorts AS (
    SELECT
        user_id,
        signup_week,
        activity_week,
        is_active
    FROM
        user_activity
),
cohort_buckets AS (
    SELECT
        signup_week AS cohort_week,
        activity_week - signup_week + 1 AS retention_period
    FROM
        cohorts
),
user_counts AS (
    SELECT
        cohort_week,
        retention_period,
        COUNT(*) FILTER (WHERE is_active) AS active_users
    FROM
        cohorts
    GROUP BY
        cohort_week, retention_period
),
retention_rates AS (
    SELECT
        cohort_week,
        retention_period,
        COUNT(DISTINCT user_id) OVER (PARTITION BY cohort_week) AS total_users,
        active_users::float / COUNT(DISTINCT user_id) OVER (PARTITION BY cohort_week) * 100 AS retention_rate
    FROM
        user_counts
),
heatmap_data AS (
    SELECT
        cohort_week - retention_period + 1 AS week_index,
        retention_period,
        COALESCE(retention_rate, 0) AS rate
    FROM
        retention_rates
)
SELECT
    week_index,
    array_agg(rate ORDER BY retention_period DESC) AS heatmap_row
FROM
    heatmap_data
GROUP BY
    week_index
ORDER BY
    week_index;

#### Closing Summary
+ Common Mistakes/Follow-Ups

The SQL solution efficiently computes cohort retention and formats the data for a triangle heatmap. The key steps involve creating cohorts, counting active users, computing rates, and reshaping the data.

#### Common Mistakes

Misinterpreting the cohort week or retention period.
Incorrectly handling NULL values in rate calculations.
Failing to ensure proper ordering of rows for heatmap visualization.

For follow-ups, you might explore:

Handling Edge Cases: What if a user signs up and never logs in?

#### Performance Considerations
How can this query be optimized with large datasets?
Visualization Enhancements: Adding additional context like total users or engagement metrics to the heatmap.

**Code:**

```sql
WITH cohorts AS (
    SELECT
        user_id,
        signup_week,
        activity_week,
        is_active
    FROM
        user_activity
),
cohort_buckets AS (
    SELECT
        signup_week AS cohort_week,
        activity_week - signup_week + 1 AS retention_period
    FROM
        cohorts
),
user_counts AS (
    SELECT
        cohort_week,
        retention_period,
        COUNT(*) FILTER (WHERE is_active) AS active_users
    FROM
        cohorts
    GROUP BY
        cohort_week, retention_period
),
retention_rates AS (
    SELECT
        cohort_week,
        retention_period,
        COUNT(DISTINCT user_id) OVER (PARTITION BY cohort_week) AS total_users,
        active_users::float / COUNT(DISTINCT user_id) OVER (PARTITION BY cohort_week) * 100 AS retention_rate
    FROM
        user_counts
),
heatmap_data AS (
    SELECT
        cohort_week - retention_period + 1 AS week_index,
        retention_period,
        COALESCE(retention_rate, 0) AS rate
    FROM
        retention_rates
)
SELECT
    week_index,
    array_agg(rate ORDER BY retention_period DESC) AS heatmap_row
FROM
    heatmap_data
GROUP BY
    week_index
ORDER BY
    week_index;
```

**Example input / output:**

```text
No specific input-output example provided, but the query above will produce a result set that can be used to create a triangle heatmap directly by the frontend.
```

