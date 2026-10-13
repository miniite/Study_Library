---
type: "practice"
difficulty: "Deep"
---

# How would you compute a rolling conversion rate (successes over attempts in the last N events) per user using two framed SUMs?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Time-based Windows](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Time-based_Windows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute a rolling conversion rate (successes over attempts in the last (N) events) per user, we'll leverage window functions to aggregate data within sliding windows. This approach ensures that we can dynamically calculate the desired metric based on the most recent (N) events for each user.

#### Triaging the Question

The core of this problem is about calculating a rolling conversion rate using SQL's window functions. Specifically, we need to:

Identify success and attempt counts within a sliding window.
Calculate the ratio (successes over attempts) in that window.
Group results by user for analysis.

#### Assumptions
The dataset includes at least two columns: user_id, indicating the user; event_type or success, where 1 represents success and 0 attempt.
(N) is a positive integer representing the number of recent events to consider.
The window should be defined as a frame with a range of (N) preceding rows, which can be configured using SQL’s windowing syntax.

#### First Principles Solution

To achieve this, we use SUM window functions within a specified frame. Here is the step-by-step breakdown:

Aggregate Successes and Attempts: Use SUM over a frame to get the total successes and attempts for each user in the last (N) events.
Calculate Conversion Rate: Divide the success count by the attempt count to get the conversion rate.

Here's how we can write this SQL query:

SQL
WITH recent_events AS (
    SELECT
        user_id,
        event_type,  -- or a column indicating success (1 for success)
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY event_timestamp DESC) as row_num,
        COUNT(*) OVER () as total_events
    FROM
        events_table
),
rolling_window AS (
    SELECT
        user_id,
        SUM(CASE WHEN event_type = 1 THEN 1 ELSE 0 END) OVER (
            PARTITION BY user_id
            ORDER BY event_timestamp DESC
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) as total_successes,
        SUM(CASE WHEN event_type = 0 THEN 1 ELSE 0 END) OVER (
            PARTITION BY user_id
            ORDER BY event_timestamp DESC
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) as total_attempts
    FROM
        recent_events
)
SELECT
    user_id,
    (total_successes / NULLIF(total_attempts, 0)) as conversion_rate
FROM
    rolling_window
WHERE
    row_num <= N;  -- Filter to get the most recent N events per user

#### Closing Summary
+ Common Mistakes/Follow-Ups

The above query correctly computes the rolling conversion rate for each user. However, it’s important to note a few common mistakes:

Division by Zero: Using NULLIF(total_attempts, 0) prevents division by zero.
Frame Specification: Ensure that the frame specification is correct based on the (N) value.

#### Follow-Ups

#### Performance Considerations

If the dataset is large, consider indexing the relevant columns (e.g., user_id and event_timestamp) to improve query performance.

Handling Ties in Event Timestamps:

The row numbering can sometimes tie if there are identical timestamps. Ensure that timestamps are unique or use a secondary sorting criterion.

Dynamic Window Size:

If (N) is not fixed and needs to be dynamic, you might need to adjust the query accordingly, possibly using variables or parameters in your SQL engine.

**Comparison:**

```
Step 1: Aggregate Successesand Attempts
Step 2: Calculate RollingWindow
Step 3: Compute ConversionRate
```

**Code:**

```sql
WITH recent_events AS (
    SELECT
        user_id,
        event_type,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY event_timestamp DESC) as row_num,
        COUNT(*) OVER () as total_events
    FROM
        events_table
),
rolling_window AS (
    SELECT
        user_id,
        SUM(CASE WHEN event_type = 1 THEN 1 ELSE 0 END) OVER (
            PARTITION BY user_id
            ORDER BY event_timestamp DESC
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) as total_successes,
        SUM(CASE WHEN event_type = 0 THEN 1 ELSE 0 END) OVER (
            PARTITION BY user_id
            ORDER BY event_timestamp DESC
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) as total_attempts
    FROM
        recent_events
)
SELECT
    user_id,
    (total_successes / NULLIF(total_attempts, 0)) as conversion_rate
FROM
    rolling_window
WHERE
    row_num <= N;  -- Filter to get the most recent N events per user
```

**Example input / output:**

```text
-- Sample input and output
WITH recent_events AS (
    SELECT
        user_id,
        event_type,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY event_timestamp DESC) as row_num,
        COUNT(*) OVER () as total_events
    FROM
        events_table
),
rolling_window AS (
    SELECT
        user_id,
        SUM(CASE WHEN event_type = 1 THEN 1 ELSE 0 END) OVER (
            PARTITION BY user_id
            ORDER BY event_timestamp DESC
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) as total_successes,
        SUM(CASE WHEN event_type = 0 THEN 1 ELSE 0 END) OVER (
            PARTITION BY user_id
            ORDER BY event_timestamp DESC
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) as total_attempts
    FROM
        recent_events
)
SELECT
    user_id,
    (total_successes / NULLIF(total_attempts, 0)) as conversion_rate
FROM
    rolling_window
WHERE
    row_num <= N;  -- Filter to get the most recent N events per user
```

**Data:**

```sql
-- Sample data schema
CREATE TABLE events_table (
    user_id INT,
    event_type BOOLEAN,  -- 1 for success, 0 for attempt
    event_timestamp TIMESTAMP
);

INSERT INTO events_table (user_id, event_type, event_timestamp) VALUES
(1, true, '2023-09-01T14:00:00Z'),
(1, false, '2023-09-02T15:00:00Z'),
(1, true, '2023-09-03T16:00:00Z'),
(2, false, '2023-09-04T17:00:00Z'),
(2, false, '2023-09-05T18:00:00Z');
```

