---
type: "practice"
difficulty: "Intermediate"
---

# What SQL would you write to create a binary label column: 1 if a user churned (no activity in 30 days), else 0?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Retention Queries](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Retention_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, we need to write an SQL query that identifies users who have not had any activity for at least 30 consecutive days and then labels them with a binary value of 1 (churned) or 0 (not churned). The key steps are filtering users based on their last activity date and comparing it against the current date, then assigning a label accordingly.

#### Triaging the Question

The question is asking for an SQL query to create a new column that classifies users as either churned (1) if they have no activity in the past 30 days or not churned (0) otherwise. The main points to consider are:

Understanding the schema of the user and activity tables.
Identifying the last activity date for each user.
Calculating the difference between the current date and the last activity date.
Assigning a label based on whether this difference exceeds 30 days.

#### First Principles Solution
Identify the Last Activity Date: We need to find the latest (most recent) activity date for each user.
Calculate the Difference in Days: Subtract the current date from the last activity date and check if it is greater than or equal to 30 days.
Assign Labels Based on the Condition.
SQL Query

Here's how you can achieve this using a common table expression (CTE) and a CASE statement:

SQL
WITH LatestActivities AS (
    SELECT
        user_id,
        MAX(activity_date) AS last_activity_date
    FROM
        activities
    GROUP BY
        user_id
)
SELECT
    a.user_id,
    a.activity_date,
    CASE
        WHEN datediff(CURRENT_DATE, la.last_activity_date) >= 30 THEN 1
        ELSE 0
    END AS churn_label
FROM
    activities a
JOIN
    LatestActivities la ON a.user_id = la.user_id;
Explanation
LatestActivities CTE: This CTE finds the latest activity date for each user.
Main Query: It joins this CTE with the activities table to get all details and then uses a CASE statement to assign labels. The datediff function calculates the difference in days between the current date and the last activity date.

#### Closing Summary

The SQL query effectively creates a binary label column based on user inactivity over a 30-day period by joining the activities table with a CTE that identifies the latest activity for each user, then applying a condition to determine churn status.

#### Common Mistakes/Follow-ups
Incorrect Date Calculation: Ensure you are using the correct date function (e.g., datediff in SQL Server or similar functions in other databases).
Missing Joins: Double-check that all necessary tables and relationships are correctly joined.
Handling NULL Values: Consider users who might not have any activities recorded, which would require a slight modification to handle such cases appropriately.

**Comparison:**

```
tableLatestActivities = [
    ["user_id", "last_activity_date"],
    [1, "2023-10-05"],
    [2, "2023-10-20"],
    [3, null],
    [4, "2023-10-25"]
]

tableActivities = [
    ["user_id", "activity_date"],
    [1, "2023-10-05"],
    [2, "2023-10-20"],
    [3, null],
    [4, "2023-10-25"]
]

tableOutput = [
    ["user_id", "activity_date", "churn_label"],
    [1, "2023-10-05", 0],
    [2, "2023-10-20", 0],
    [3, null, 1],
    [4, "2023-10-25", 0]
]
```

**Code:**

```sql
WITH LatestActivities AS (
    SELECT
        user_id,
        MAX(activity_date) AS last_activity_date
    FROM
        activities
    GROUP BY
        user_id
)
SELECT
    a.user_id,
    a.activity_date,
    CASE
        WHEN datediff(CURRENT_DATE, la.last_activity_date) >= 30 THEN 1
        ELSE 0
    END AS churn_label
FROM
    activities a
JOIN
    LatestActivities la ON a.user_id = la.user_id;
```

**Example input / output:**

| user_id | activity_date | churn_label |
|---------|---------------|-------------|
| 1       | 2023-10-05    | 0           |
| 2       | 2023-10-20    | 0           |
| 3       | null          | 1 (NULL)    |
| 4       | 2023-10-25    | 0           |

**Data:**

```json
{
  "activities": [
    {"user_id": 1, "activity_date": "2023-10-05"},
    {"user_id": 2, "activity_date": "2023-10-20"},
    {"user_id": 3, "activity_date": null},
    {"user_id": 4, "activity_date": "2023-10-25"}
  ]
}
```

**Flow:**

1. No
2. Yes
3. Identify Latest Activity Date
4. Is Last Activity > 30 Days?
5. Label as Not Churned (0)
6. Label as Churned (1)

