---
type: "practice"
difficulty: "Deep"
---

# How would you write a query to compute a weekly cohort of new signups for a retention analysis feeding a retention model?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Retention Queries](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Retention_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute a weekly cohort of new signups, we need to identify users who signed up in each week and then track their activity or inactivity over time. This will help us analyze user retention patterns effectively.

#### Triage

The interviewer is probing your ability to:

Understand the concept of cohorts.
Write an SQL query that can handle temporal data.

#### Cons
ider edge cases such as different date formats, timezone differences, and handling null values.

#### Assumptions

The sign-up date for each user is stored in a column named signup_date.
The dataset contains multiple records per user, with the latest signup_date being the primary one used to form cohorts.
There are no duplicate entries for the same user on different dates.

#### First Principles Solution
Identify Weekly Cohorts:
We need to determine in which week each user signed up. This can be done by converting the signup date into a week number.
Track User Activity:
For simplicity, we'll track whether a user is still active or has churned after 30 days from their sign-up date.
SQL Query

Below is an example of how you could write this query:

SQL
WITH weekly_signups AS (
    SELECT
        user_id,
        signup_date,
        DATE_TRUNC('week', signup_date) AS week_of_signup
    FROM
        users_table
),
user_activity AS (
    SELECT
        user_id,
        signup_date,
        signup_week := DATE_TRUNC('week', signup_date) AS week_of_signup,
        latest_event_date := MAX(event_date) OVER (PARTITION BY user_id) AS latest_event_date
    FROM
        users_table
)
SELECT
    ws.week_of_signup AS week_number,
    COUNT(ws.user_id) AS new_signups,
    COUNT(CASE WHEN wa.latest_event_date < DATEADD(day, 30, wa.signup_week) THEN 1 ELSE NULL END) AS churned_users
FROM
    weekly_signups ws
LEFT JOIN
    user_activity wa ON ws.user_id = wa.user_id AND ws.week_of_signup = wa.week_of_signup
GROUP BY
    ws.week_of_signup;

#### Closing Summary
+ Common Mistakes/Follow-Ups
Common Mistake: Forgetting to convert the signup date into a week number, which could lead to incorrect cohort assignments.

#### Follow-Up
Question: Discuss how you would handle users who sign up on the same day but have different times (e.g., timezone differences).
Advanced Consideration: If there are millions of records, consider optimizing the query using indexes or partitioning.

**Comparison:**

| week_number | new_signups | churned_users |
|-------------|-------------|---------------|
| 2023-W01    | 5           | 4             |

**Code:**

```sql
WITH weekly_signups AS (
    SELECT
        user_id,
        signup_date,
        DATE_TRUNC('week', signup_date) AS week_of_signup
    FROM
        users_table
),
user_activity AS (
    SELECT
        user_id,
        signup_date,
        signup_week := DATE_TRUNC('week', signup_date) AS week_of_signup,
        latest_event_date := MAX(event_date) OVER (PARTITION BY user_id) AS latest_event_date
    FROM
        users_table
)
SELECT
    ws.week_of_signup AS week_number,
    COUNT(ws.user_id) AS new_signups,
    COUNT(CASE WHEN wa.latest_event_date < DATEADD(day, 30, wa.signup_week) THEN 1 ELSE NULL END) AS churned_users
FROM
    weekly_signups ws
LEFT JOIN
    user_activity wa ON ws.user_id = wa.user_id AND ws.week_of_signup = wa.week_of_signup
GROUP BY
    ws.week_of_signup;
```

**Example input / output:**

| week_number | new_signups | churned_users |
|-------------|-------------|---------------|
| 2023-W01    | 5           | 4             |

**Data:**

```json
{
  "users_table": {
    "user_id": [1, 2, 3, 4, 5],
    "signup_date": ["2023-01-02", "2023-01-05", "2023-01-08", "2023-01-10", "2023-01-12"],
    "event_date": ["2023-01-15", null, null, "2023-01-20", null]
  }
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. erDiagram
3. users_table ||--o{ weekly_signups : has_weekly_signup
4. users_table ||--o{ user_activity : has_user_activity
5. weekly_signups {
6. PK user_id
7. DATE signup_date
8. DATE week_of_signup
9. }
10. user_activity {
11. PK user_id
12. DATE signup_date
13. DATE latest_event_date
14. INT week_of_signup
15. }

