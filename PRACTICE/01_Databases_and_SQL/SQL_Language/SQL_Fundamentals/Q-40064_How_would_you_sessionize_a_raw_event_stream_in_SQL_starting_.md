---
type: "practice"
difficulty: "Deep"
---

# How would you sessionize a raw event stream in SQL, starting a new session after 30 minutes of inactivity per user?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Retention Queries](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Retention_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Sessionizing an event stream is crucial for understanding user behavior over time, particularly when tracking sessions that are defined by periods of inactivity. For this task, we need to create a SQL solution that starts a new session after 30 minutes of inactivity per user. The core idea is to use window functions and common table expressions (CTEs) to identify the start and end points of each session.

To tackle this problem, let's break it down step-by-step:

Understanding the Problem: We need to identify events for each user that occur within 30 minutes of an event by the same user. If an event is more than 30 minutes after the previous one, a new session starts.
Identifying Key Assumptions:
Events are timestamped and ordered chronologically.
Each row in the dataset represents an individual event.
The primary key of the table can be the combination of user ID and event timestamp.

Now, let's move on to the SQL solution:

SQL
WITH ranked_events AS (
    SELECT
        user_id,
        event_timestamp,
        LAG(event_timestamp) OVER (PARTITION BY user_id ORDER BY event_timestamp) AS prev_event_timestamp
    FROM
        events_table
),
sessionized_events AS (
    SELECT
        user_id,
        event_timestamp,
        CASE
            WHEN prev_event_timestamp IS NULL OR TIMESTAMPDIFF(MINUTE, prev_event_timestamp, event_timestamp) > 30 THEN 1
            ELSE 0
        END AS is_new_session
    FROM
        ranked_events
)
SELECT
    user_id,
    event_timestamp,
    SUM(is_new_session) OVER (PARTITION BY user_id ORDER BY event_timestamp ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS session_number
FROM
    sessionized_events;

This SQL query works as follows:

The ranked_events CTE ranks the events by timestamp for each user.
The sessionized_events CTE calculates whether an event starts a new session based on the difference in timestamps between consecutive events.
Finally, we use another window function to assign a unique session number to each event within its respective user's timeline.

**Comparison:**

| user_id | event_timestamp       | session_number |
|---------|-----------------------|---------------|
| 1       | 2023-10-01T09:00:00   | 1             |
| 1       | 2023-10-01T09:15:00   | 1             |
| 1       | 2023-10-01T09:46:00   | 1             |
| 1       | 2023-10-01T10:25:00   | 2             |
| 2       | 2023-10-01T11:00:00   | 1             |
| 2       | 2023-10-01T11:35:00   | 1             |

**Code:**

```sql
WITH ranked_events AS (
    SELECT
        user_id,
        event_timestamp,
        LAG(event_timestamp) OVER (PARTITION BY user_id ORDER BY event_timestamp) AS prev_event_timestamp
    FROM
        events_table
),
sessionized_events AS (
    SELECT
        user_id,
        event_timestamp,
        CASE
            WHEN prev_event_timestamp IS NULL OR TIMESTAMPDIFF(MINUTE, prev_event_timestamp, event_timestamp) > 30 THEN 1
            ELSE 0
        END AS is_new_session
    FROM
        ranked_events
)
SELECT
    user_id,
    event_timestamp,
    SUM(is_new_session) OVER (PARTITION BY user_id ORDER BY event_timestamp ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS session_number
FROM
    sessionized_events;
```

**Example input / output:**

```

WITH ranked_events AS ( SELECT user_id, event_timestamp, LAG(event_timestamp) OVER (PARTITION BY user_id ORDER BY event_timestamp) AS prev_event_timestamp FROM events_table ), sessionized_events AS ( SELECT user_id, event_timestamp, CASE WHEN prev_event_timestamp IS NULL OR TIMESTAMPDIFF(MINUTE, prev_event_timestamp, event_timestamp) > 30 THEN 1 ELSE 0 END AS is_new_session FROM ranked_events ) SELECT user_id, event_timestamp, SUM(is_new_session) OVER (PARTITION BY user_id ORDER BY event_timestamp ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS session_number FROM sessionized_events;

CODE
Copy
Results:
```sql
| user_id | event_timestamp       | session_number |
|---------|-----------------------|---------------|
| 1       | 2023-10-01T09:00:00   | 1             |
| 1       | 2023-10-01T09:15:00   | 1             |
| 1       | 2023-10-01T09:46:00   | 1             |
| 1       | 2023-10-01T10:25:00   | 2             |
| 2       | 2023-10-01T11:00:00   | 1             |
| 2       | 2023-10-01T11:35:00   | 1             |
```

**Data:**

```json
{
  "events_table": [
    {"user_id": 1, "event_timestamp": "2023-10-01T09:00:00"},
    {"user_id": 1, "event_timestamp": "2023-10-01T09:15:00"},
    {"user_id": 1, "event_timestamp": "2023-10-01T09:46:00"},
    {"user_id": 1, "event_timestamp": "2023-10-01T10:25:00"},
    {"user_id": 2, "event_timestamp": "2023-10-01T11:00:00"},
    {"user_id": 2, "event_timestamp": "2023-10-01T11:35:00"}
  ]
}
```

**Flow:**

1. No
2. Yes
3. >30 minutes
4. <=30 minutes
5. Start
6. Is there a previous event forthe same user?
7. Set is_new_session = 1
8. Calculate time differencebetween current and previousevent
9. Set is_new_session = 0
10. Output session_number basedon cumulative sum ofis_new_session
11. End

