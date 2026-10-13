---
type: "practice"
difficulty: "Deep"
---

# How would you compute retention: the number of users active in month N who were also active in month N minus 1?

**Difficulty:** Deep · **Levels:** Fresher · [↑ Classic SQL Query Problems](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Classic_SQL_Query_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To calculate user retention, we need to determine how many users from one month are still active in the next month. This involves comparing two snapshots of user activity data—typically a week or a month apart. Let’s break down the process step by step.

#### Triaging the Question

The question is asking us to compute user retention over a specific time period. The core idea here is to identify users who were active in one month and then check if they are still active in the next month. We will need two data points: the number of active users in month N-1 (the baseline) and the number of active users in month N (the current snapshot).

#### Assumptions
User Activity Data: We have a table that records user activity, likely with columns for user_id and activity_date.
Time Window: The time window is typically one month, but could be adjusted based on business needs.
Active Users: An active user can be defined as someone who performed an action (e.g., logged in, made a purchase) within the specified period.

#### First-Principles Solution

Data Preparation:

Extract data for month N-1 and month N.
Ensure both datasets are clean and consistent.

Identify Active Users:

For each dataset, identify users who were active in their respective months by checking the presence of activity records.

Joining Data:

Perform a left join on user_id between month N-1 and month N datasets to find matching users across both periods.

Count Active Users:

Filter out non-matching rows (users who are active in one month but not the other) to get only those users who were present in both months.
Count these filtered records to determine retention.
Detailed Steps
Extract and Clean Data:
SQL
-- SQL Example for Extracting Month N-1 and Month N Data
WITH
    month_n_minus_1_data AS (
        SELECT user_id, activity_date FROM user_activity WHERE EXTRACT(MONTH FROM activity_date) = (CURRENT_MONTH - 1)
    ),
    month_n_data AS (
        SELECT user_id, activity_date FROM user_activity WHERE EXTRACT(MONTH FROM activity_date) = CURRENT_MONTH
    )
Identify Active Users:
SQL
-- Identifying active users in each period
active_month_n_minus_1 AS (
    SELECT DISTINCT user_id FROM month_n_minus_1_data
),
active_month_n AS (
    SELECT DISTINCT user_id FROM month_n_data
)
Joining Data:
SQL
-- Joining the datasets on user_id to find common users
JOINED_DATA AS (
    SELECT a.user_id
    FROM active_month_n_minus_1 a
    LEFT JOIN active_month_n b ON a.user_id = b.user_id
)
Count Active Users in Both Periods:
SQL
-- Filtering and counting the retained users
SELECT COUNT(*) FROM JOINED_DATA WHERE user_id IS NOT NULL;

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, to compute user retention, we need to:

Extract clean activity data for two consecutive months.
Identify active users in each month.
Join the datasets on user_id to find common users.
Count these users as they represent retained users.

#### Common mistakes
include:

Failing to properly clean and filter the input data, leading to incorrect counts.
Misunderstanding the definition of "activity" or failing to adjust for different types of user activities (e.g., login vs. transaction).
Not considering edge cases where the time window is not aligned with calendar months.

#### Follow-ups
could include:

#### Cons
idering additional metrics like new users and churn rate.
Implementing this logic in a real-time streaming system if needed.
Optimizing performance for large datasets.

**Comparison:**

```
Month N-1 Data
Select DISTINCT user_id
Month N Data
Select DISTINCT user_id
JOIN ON user_id
Filter user_id IS NOT NULL
```

**Code:**

```sql
WITH
    month_n_minus_1_data AS (
        SELECT user_id, activity_date FROM user_activity WHERE EXTRACT(MONTH FROM activity_date) = (CURRENT_MONTH - 1)
    ),
    month_n_data AS (
        SELECT user_id, activity_date FROM user_activity WHERE EXTRACT(MONTH FROM activity_date) = CURRENT_MONTH
    ),
    active_month_n_minus_1 AS (
        SELECT DISTINCT user_id FROM month_n_minus_1_data
    ),
    active_month_n AS (
        SELECT DISTINCT user_id FROM month_n_data
    ),
    JOINED_DATA AS (
        SELECT a.user_id
        FROM active_month_n_minus_1 a
        LEFT JOIN active_month_n b ON a.user_id = b.user_id
    )
SELECT COUNT(*) FROM JOINED_DATA WHERE user_id IS NOT NULL;
```

**Example input / output:**

```text
WITH
    month_n_minus_1_data AS (
        SELECT user_id, activity_date FROM user_activity WHERE EXTRACT(MONTH FROM activity_date) = (CURRENT_MONTH - 1)
    ),
    month_n_data AS (
        SELECT user_id, activity_date FROM user_activity WHERE EXTRACT(MONTH FROM activity_date) = CURRENT_MONTH
    ),
    active_month_n_minus_1 AS (
        SELECT DISTINCT user_id FROM month_n_minus_1_data
    ),
    active_month_n AS (
        SELECT DISTINCT user_id FROM month_n_data
    ),
    JOINED_DATA AS (
        SELECT a.user_id
        FROM active_month_n_minus_1 a
        LEFT JOIN active_month_n b ON a.user_id = b.user_id
    )
SELECT COUNT(*) FROM JOINED_DATA WHERE user_id IS NOT NULL;
```

**Data:**

```json
{
  "user_activity": [
    { "user_id": 1, "activity_date": "2023-10-05" },
    { "user_id": 2, "activity_date": "2023-10-06" },
    { "user_id": 3, "activity_date": "2023-10-07" },
    { "user_id": 4, "activity_date": "2023-10-08" },
    { "user_id": 5, "activity_date": "2023-10-09" }
  ]
}
```

**Flow:**

1. EXTRACT(MONTH FROMactivity_date) =(CURRENT_MONTH - 1)
2. EXTRACT(MONTH FROMactivity_date) =CURRENT_MONTH
3. Month N-1 Data
4. Select DISTINCT user_id
5. Month N Data
6. Select DISTINCT user_id
7. JOIN ON user_id
8. Filter user_id IS NOT NULL

