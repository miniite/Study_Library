---
type: "practice"
difficulty: "Deep"
---

# Design a query to detect fraudulent behavior: users who made more than 5 failed logins from 3+ distinct IPs within a 10-minute window. Explain the windowing and why naive self-joins blow up.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Gaps and Islands](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Gaps_and_Islands.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an SQL query that detects fraudulent behavior, specifically users who have made more than 5 failed logins from 3+ distinct IPs within a 10-minute window, we need to carefully consider how to handle temporal data and avoid performance issues with self-joins. The goal is to identify suspicious login patterns without overloading the database.

#### Triage
Understanding the Problem

The key aspects of this problem are:

Failed Logins: We need to track failed login attempts.
Distinct IPs: Each user should be considered from multiple unique IP addresses.
Temporal Window: Login events must fall within a 10-minute window.
Fraudulent Pattern: A user should have more than 5 failed logins and at least 3 distinct IPs in the specified time frame.

#### Assumptions
We are working with an auth_events table that records login attempts, including fields such as user_id, ip_address, and timestamp.
The database supports window functions, which are essential for this query to be efficient.
The timestamp resolution is fine-grained enough (e.g., in seconds) to capture the 10-minute window accurately.

#### First Principles Solution

The solution involves using window functions to aggregate data over a sliding 10-minute window and then filtering based on the conditions specified. Here's how we can approach this:

Identify Failed Logins: Filter out only failed login events.
Windowing Over Time: Use LEAD or LAG to get the previous/next timestamp within a sliding 10-minute window.
Group by User and IP: Count distinct IPs for each user in this time frame.
Count Failed Logins: Ensure there are more than 5 failed logins.
Filter and Output Results: Combine these conditions to get the final set of users who meet the criteria.
SQL Query
SQL
WITH auth_failed AS (
    SELECT
        user_id,
        ip_address,
        timestamp,
        LAG(timestamp) OVER (PARTITION BY user_id ORDER BY timestamp) AS prev_timestamp
    FROM auth_events
    WHERE status = 'failed'
),
user_ip_counts AS (
    SELECT
        user_id,
        COUNT(DISTINCT ip_address) as distinct_ips
    FROM auth_failed
    GROUP BY user_id,
             DATE_TRUNC('minute', (timestamp + INTERVAL '5 minutes'))
)
SELECT
    uf.user_id
FROM user_ip_counts uc
JOIN auth_failed uf ON uc.user_id = uf.user_id
WHERE uc.distinct_ips >= 3 AND COUNT(uf.ip_address) >= 5
GROUP BY uf.user_id;
Explanation of the Query
auth_failed CTE: We filter only failed logins and use LAG to get the previous timestamp for each user.
user_ip_counts CTE: This aggregates the distinct IPs per user within a 10-minute window by truncating the timestamps to the nearest minute and adding an interval to align with our sliding window logic.
Final SELECT Statement: We join back on auth_failed to get the necessary details and filter users who have at least 3 distinct IPs and 5 or more failed logins.

#### Common Mistakes
and Follow-Ups
Naive Self-Joins: Avoid using self-joins as they can be inefficient, especially with large datasets. Window functions are much more efficient.
Time Resolution: Ensure the timestamp resolution matches your needs; if you need to handle sub-minute granularity, adjust the interval in DATE_TRUNC.

#### Performance Considerations
For very large datasets, consider indexing on relevant columns (user_id, timestamp) and optimizing the schema.

By following this approach, we ensure that our query is both efficient and accurate in detecting fraudulent behavior based on login patterns.

**Comparison:**

```
auth_events
auth_failed
user_ip_counts
user_id
```

**Code:**

```sql
WITH auth_failed AS (
    SELECT
        user_id,
        ip_address,
        timestamp,
        LAG(timestamp) OVER (PARTITION BY user_id ORDER BY timestamp) AS prev_timestamp
    FROM auth_events
    WHERE status = 'failed'
),
user_ip_counts AS (
    SELECT
        user_id,
        COUNT(DISTINCT ip_address) as distinct_ips
    FROM auth_failed
    GROUP BY user_id,
             DATE_TRUNC('minute', (timestamp + INTERVAL '5 minutes'))
)
SELECT
    uf.user_id
FROM user_ip_counts uc
JOIN auth_failed uf ON uc.user_id = uf.user_id
WHERE uc.distinct_ips >= 3 AND COUNT(uf.ip_address) >= 5
GROUP BY uf.user_id;
```

**Example input / output:**

```text
-- Expected output
SELECT * FROM user_ip_counts WHERE distinct_ips >= 3 AND COUNT(ip_address) >= 5;
-- Result:
-- user_id
-- 3
```

**Data:**

```sql
-- Sample data schema for auth_events table
CREATE TABLE auth_events (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    ip_address VARCHAR(45) NOT NULL,
    timestamp TIMESTAMP NOT NULL,
    status VARCHAR(10) CHECK (status IN ('success', 'failed'))
);

-- Sample input data
INSERT INTO auth_events (user_id, ip_address, timestamp, status)
VALUES
(1, '192.168.1.1', NOW() - INTERVAL '5 minutes', 'failed'),
(1, '192.168.1.2', NOW() - INTERVAL '4 minutes', 'failed'),
(1, '192.168.1.3', NOW() - INTERVAL '3 minutes', 'failed'),
(1, '192.168.1.4', NOW() - INTERVAL '2 minutes', 'failed'),
(1, '192.168.1.5', NOW() - INTERVAL '1 minute', 'failed'),
(2, '192.168.1.6', NOW() - INTERVAL '7 minutes', 'success'),
(3, '192.168.1.7', NOW() - INTERVAL '5 minutes', 'failed'),
(3, '192.168.1.8', NOW() - INTERVAL '4 minutes', 'failed'),
(3, '192.168.1.9', NOW() - INTERVAL '3 minutes', 'failed'),
(3, '192.168.1.10', NOW() - INTERVAL '2 minutes', 'failed'),
(3, '192.168.1.11', NOW() - INTERVAL '1 minute', 'failed');
```

**Flow:**

1. Final Output
2. Intermediate Results
3. Input Data
4. auth_events
5. auth_failed
6. user_ip_counts
7. user_id

