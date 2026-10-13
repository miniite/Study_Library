---
type: "practice"
difficulty: "Intermediate"
---

# How would you design a query to compute daily new versus returning users from an events table?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Classic SQL Query Problems](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Classic_SQL_Query_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a query that computes daily new versus returning users, we first need to understand what constitutes "new" and "returning" users based on their event data. Generally, new users might not have any previous interaction with the system, while returning users are those who interacted at least once before.

The events table would likely contain user identifiers (e.g., user_id) along with a timestamp or date of each event (event_date). Here, we need to distinguish between these two categories for daily analysis. Let’s outline our solution step-by-step.

**Comparison:**

| user_id | first_event_date | days_since_first_event |
|---------|------------------|-----------------------|
| 1       | 2023-01-01       | 4                     |
| 2       | 2023-01-02       | 2                     |
| 3       | 2023-01-06       | 0                     |

**Code:**

```sql
-- Step 1: Identify new users per day by checking if they have no previous interaction on earlier days

WITH daily_interactions AS (
    SELECT
        user_id,
        event_date,
        LAG(event_date) OVER (PARTITION BY user_id ORDER BY event_date) AS prev_event_date
    FROM events
),
first_day AS (
    SELECT DISTINCT
        user_id,
        MIN(event_date) AS first_event_date
    FROM daily_interactions
    WHERE prev_event_date IS NULL -- first interaction with the system
    GROUP BY user_id
)
SELECT
    fd.user_id,
    fd.first_event_date,
    ed.event_date - fd.first_event_date AS days_since_first_event
FROM first_day fd
JOIN daily_interactions ed ON fd.user_id = ed.user_id AND fd.first_event_date = ed.event_date;
```

**Example input / output:**

```

Given the sample data:

JSON
Copy
[
  {"user_id": 1, "event_date": "2023-01-01"},
  {"user_id": 1, "event_date": "2023-01-05"},
  {"user_id": 2, "event_date": "2023-01-02"},
  {"user_id": 2, "event_date": "2023-01-04"},
  {"user_id": 3, "event_date": "2023-01-06"}
]

The output will be:

JSON
Copy
[
  {"user_id": 1, "first_event_date": "2023-01-01", "days_since_first_event": 4},
  {"user_id": 2, "first_event_date": "2023-01-02", "days_since_first_event": 2},
  {"user_id": 3, "first_event_date": "2023-01-06", "days_since_first_event": 0}
]

This shows the first day each user interacted and how many days passed since their initial interaction.
```

**Data:**

```json
{
  "events": [
    {"user_id": 1, "event_date": "2023-01-01"},
    {"user_id": 1, "event_date": "2023-01-05"},
    {"user_id": 2, "event_date": "2023-01-02"},
    {"user_id": 2, "event_date": "2023-01-04"},
    {"user_id": 3, "event_date": "2023-01-06"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Is user_id's first interaction onthe same date?
7. Mark as new user
8. Check if previous event existsfor this user?
9. Incrementdays_since_first_event counter
10. Mark as returning user, resetdays_since_first_event to 0
11. Continue with next row
12. End

