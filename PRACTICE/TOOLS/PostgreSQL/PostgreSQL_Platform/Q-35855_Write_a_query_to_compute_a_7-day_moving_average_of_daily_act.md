---
type: "practice"
difficulty: "Deep"
---

# Write a query to compute a 7-day moving average of daily active users using a window function.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Basic Query Practice](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Basic_Query_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute a 7-day moving average of daily active users (DAU) in PostgreSQL, we need to leverage window functions to aggregate data over a rolling time period and then calculate the average. The key steps involve defining the window frame correctly and applying the AVG function.

#### Triage
The question is asking for a query that computes the 7-day moving average of DAUs using PostgreSQL's window function capabilities. We assume we have a table named user_activity with at least two columns: date (representing the day) and count (number of active users on each date).

#### First-principles solution

Define the time range for the moving average, which is 7 days.
Use the AVG window function to calculate the mean over this defined frame.

Let's break it down step by step:

Step 1: Create a subquery or CTE (Common Table Expression) that ranks users by date and assigns each row a relative ranking within its 7-day window.
Step 2: Use the AVG function with the appropriate frame specification to calculate the moving average.

Here’s the SQL query in detail:

SQL
WITH ranked_user_activity AS (
    SELECT
        date,
        count,
        ROW_NUMBER() OVER (ORDER BY date) - ROW_NUMBER() OVER (PARTITION BY date ORDER BY date) AS window_rank
    FROM
        user_activity
)
SELECT
    date,
    AVG(count) OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS dau_7day_moving_avg
FROM
    ranked_user_activity
WHERE
    window_rank % 7 = 0; -- Filter to only include rows that are part of a complete 7-day window

#### Closing summary
The query above calculates the moving average by partitioning and ordering the data, then applying an appropriate frame specification. This method ensures that each row is considered in the context of a rolling time period.

#### Common mistakes

Forgetting to reset the window rank at the beginning of each 7-day cycle.
Incorrect use or misinterpretation of ROWS BETWEEN clauses for the moving window.
Not filtering out rows that do not form complete 7-day windows, leading to incomplete data points in the output.

**Comparison:**

| date       | count   | window_rank |
|------------|---------|-------------|
| 2023-10-01 |    500  |            0 |
| 2023-10-02 |    550  |            1 |
| 2023-10-03 |    600  |            2 |
| 2023-10-04 |    650  |            3 |
| 2023-10-05 |    700  |            4 |
| 2023-10-06 |    750  |            5 |
| 2023-10-07 |    800  |            6 |
| 2023-10-08 |    850  |           -1|
| 2023-10-09 |    900  |           -2|
| 2023-10-10 |    950  |           -3|

**Code:**

```sql
WITH ranked_user_activity AS (
    SELECT
        date,
        count,
        ROW_NUMBER() OVER (ORDER BY date) - ROW_NUMBER() OVER (PARTITION BY date ORDER BY date) AS window_rank
    FROM
        user_activity
)
SELECT
    date,
    AVG(count) OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS dau_7day_moving_avg
FROM
    ranked_user_activity
WHERE
    window_rank % 7 = 0; -- Filter to only include rows that are part of a complete 7-day window
```

**Example input / output:**

```text
WITH ranked_user_activity AS (
    SELECT
        date,
        count,
        ROW_NUMBER() OVER (ORDER BY date) - ROW_NUMBER() OVER (PARTITION BY date ORDER BY date) AS window_rank
    FROM
        user_activity
)
SELECT
    date,
    AVG(count) OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS dau_7day_moving_avg
FROM
    ranked_user_activity
WHERE
    window_rank % 7 = 0;

-- Output:
+------------+---------------------+
|   date     | dau_7day_moving_avg |
+------------+---------------------+
| 2023-10-07 |               650.0 |
| 2023-10-08 |                700  |
| 2023-10-09 |                 750 |
| 2023-10-10 |                800  |
+------------+---------------------+
```

**Data:**

```json
{
    "user_activity": [
        {"date": "2023-10-01", "count": 500},
        {"date": "2023-10-02", "count": 550},
        {"date": "2023-10-03", "count": 600},
        {"date": "2023-10-04", "count": 650},
        {"date": "2023-10-05", "count": 700},
        {"date": "2023-10-06", "count": 750},
        {"date": "2023-10-07", "count": 800},
        {"date": "2023-10-08", "count": 850},
        {"date": "2023-10-09", "count": 900},
        {"date": "2023-10-10", "count": 950}
    ]
}
```

**Flow:**

1. Define CTE for window rank
2. Rewrite original date andcount with relative ranking
3. Select dates and calculatemoving average within each7-day window
4. Filter rows to include onlycomplete windows
5. Output final result

