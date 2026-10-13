---
type: "practice"
difficulty: "Basic"
---

# How would you compute the number of distinct days a user was active in the last 30 days as an engagement feature?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Retention Queries](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Retention_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute the number of distinct days a user was active in the last 30 days, we can leverage SQL to extract and count unique activity dates per user. This is a common requirement in user engagement metrics where understanding user behavior over time is crucial.

The question essentially asks us to:

Identify users who have been active.
Determine their active days within the past 30 days.
Count these distinct days for each user.

#### Assumptions
we are making include:

The dataset contains at least a user_id and a timestamp or date field that captures when an activity occurred.
Active days can be inferred from non-null timestamps.

Let's break down the steps using SQL to achieve this:

Filter for the last 30 days: We need to ensure we are only considering records within the past month.
Group by user and date: This will help us count unique dates per user.
Count distinct days: For each group, we want to know how many distinct days there were.

Here is a step-by-step SQL solution:

SQL
SELECT
    user_id,
    COUNT(DISTINCT DATE(timestamp)) AS active_days_last_30_days
FROM
    user_activity_table
WHERE
    timestamp >= NOW() - INTERVAL 30 DAY
GROUP BY
    user_id;

This query does the following:

timestamp >= NOW() - INTERVAL 30 DAY: Filters records to include only those within the last 30 days.
COUNT(DISTINCT DATE(timestamp)): Counts distinct dates for each user.
GROUP BY user_id: Groups results by user to provide a count per user.

**Code:**

```sql
SELECT
    user_id,
    COUNT(DISTINCT DATE(timestamp)) AS active_days_last_30_days
FROM
    user_activity_table
WHERE
    timestamp >= NOW() - INTERVAL 30 DAY
GROUP BY
    user_id;
```

