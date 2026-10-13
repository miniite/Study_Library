---
type: "practice"
difficulty: "Deep"
---

# How would you use SQL window functions to compute per-user first-touch and last-touch attribution features, and why is a MIN/MAX subquery join subtly wrong for ties?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Attribution with Window Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Attribution_with_Window_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute per-user first-touch and last-touch attribution features using SQL window functions, we need to identify the earliest and latest timestamps of interactions for each user. The key idea here is leveraging window functions like MIN() and MAX() over a proper partitioning scheme. However, it's important to understand why a MIN/MAX subquery join can be subtly incorrect in certain scenarios.

Firstly, let’s define our dataset structure. Suppose we have an interactions table with columns: user_id, interaction_time, and possibly others like event_type. Our goal is to create two new features for each user:

The first interaction time (first-touch).
The last interaction time (last-touch).

#### Triaging the Question

The question is essentially asking how to use SQL window functions effectively to compute user-specific attributes based on timestamps. The key here is understanding the correct approach and why certain naive solutions might fail in edge cases.

#### Assumptions
We are working with a large dataset where performance considerations are important.
Each user can have multiple interactions, and we need to identify the earliest and latest times of these interactions.
There could be ties in timestamps, which add complexity to the solution.

#### Solution
from First Principles

To compute the first-touch and last-touch features, we can use window functions as follows:

Step 1: Identify Min/Max Interaction Times
SQL
WITH ranked_interactions AS (
    SELECT
        user_id,
        interaction_time,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY interaction_time) AS rn_first_touch,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY interaction_time DESC) AS rn_last_touch
    FROM interactions
)
SELECT
    user_id,
    MIN(CASE WHEN rn_first_touch = 1 THEN interaction_time ELSE NULL END) AS first_touch,
    MAX(CASE WHEN rn_last_touch = 1 THEN interaction_time ELSE NULL END) AS last_touch
FROM ranked_interactions
GROUP BY user_id;
Explanation
Window Function for Ranking: We use ROW_NUMBER() to assign a unique rank to each interaction time within each user's interactions.
Identify First Touch: By filtering rows where rn_first_touch = 1, we get the earliest timestamp for each user.
Identify Last Touch: Similarly, by filtering rows where rn_last_touch = 1, we get the latest timestamp for each user.

This approach ensures that we correctly handle ties in timestamps and provide accurate first-touch and last-touch times.

Why MIN/MAX Subquery Join is Subtle

A common mistake might be to use a subquery join like this:

SQL
SELECT
    i.user_id,
    (SELECT MIN(interaction_time) FROM interactions WHERE user_id = i.user_id) AS first_touch,
    (SELECT MAX(interaction_time) FROM interactions WHERE user_id = i.user_id) AS last_touch
FROM interactions i;

While this query looks straightforward, it has a subtle issue. If multiple users have the same minimum or maximum timestamp across all records in the table, the subquery will return the first occurrence of that timestamp for each user_id. This can lead to incorrect results if there are ties and you want to ensure the correct first and last interaction times.

#### Closing Summary

Using window functions like ROW_NUMBER() is a robust way to compute per-user first-touch and last-touch attributes. The solution ensures correctness even in edge cases where multiple users might have the same minimum or maximum timestamps.

#### Common Mistakes/Follow-Ups
Ties: Ensure your solution correctly handles ties.

#### Performance Considerations
Window functions can be efficient, but for very large datasets, indexing on user_id and interaction_time is crucial.
Edge Cases: Verify the behavior when a user has no interactions or when all timestamps are the same.

**Comparison:**

```
FLOWCHART · MERMAID
pie title User Interaction Analysis
    "First Touch" : 70%
    "Last Touch" : 30%
```

**Code:**

```sql
WITH ranked_interactions AS (
    SELECT
        user_id,
        interaction_time,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY interaction_time) AS rn_first_touch,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY interaction_time DESC) AS rn_last_touch
    FROM interactions
)
SELECT
    user_id,
    MIN(CASE WHEN rn_first_touch = 1 THEN interaction_time ELSE NULL END) AS first_touch,
    MAX(CASE WHEN rn_last_touch = 1 THEN interaction_time ELSE NULL END) AS last_touch
FROM ranked_interactions
GROUP BY user_id;
```

**Example input / output:**

```text
WITH ranked_interactions AS (
    SELECT
        user_id,
        interaction_time,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY interaction_time) AS rn_first_touch,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY interaction_time DESC) AS rn_last_touch
    FROM interactions
)
SELECT
    user_id,
    MIN(CASE WHEN rn_first_touch = 1 THEN interaction_time ELSE NULL END) AS first_touch,
    MAX(CASE WHEN rn_last_touch = 1 THEN interaction_time ELSE NULL END) AS last_touch
FROM ranked_interactions
GROUP BY user_id;

-- Output:
+----------+---------------------+---------------------+
| user_id  | first_touch         | last_touch          |
+----------+---------------------+---------------------+
| 1        | 2023-01-01T12:00:00 | 2023-01-05T18:00:00 |
| 2        | 2023-02-01T09:00:00 | 2023-02-05T14:00:00 |
| 3        | 2023-03-01T16:00:00 | 2023-03-05T21:00:00 |
+----------+---------------------+---------------------+
```

**Data:**

```json
{
  "interactions": [
    {"user_id": 1, "interaction_time": "2023-01-01T12:00:00"},
    {"user_id": 1, "interaction_time": "2023-01-05T18:00:00"},
    {"user_id": 2, "interaction_time": "2023-02-01T09:00:00"},
    {"user_id": 2, "interaction_time": "2023-02-05T14:00:00"},
    {"user_id": 3, "interaction_time": "2023-03-01T16:00:00"},
    {"user_id": 3, "interaction_time": "2023-03-05T21:00:00"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is user_id unique?
5. Compute first_touch andlast_touch using windowfunctions
6. Handle ties (optional)
7. End
8. Ensure correct handling of ties

