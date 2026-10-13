---
type: "practice"
difficulty: "Deep"
---

# Explain how you would join three tables (users, orders, sessions) to assemble a wide feature matrix without inflating row counts.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Join Fan-out](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Join_Fan-out.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve the problem of joining three tables (users, orders, and sessions) while avoiding row count inflation in a feature matrix, we need to understand that simply performing a straightforward join can result in Cartesian product-like behavior, leading to an excessive number of rows. Instead, we should aim for an inner join or appropriate filters to ensure each user is represented by the correct set of features derived from their orders and sessions.

#### Triage

The question essentially asks us to design a SQL query that efficiently combines data from three tables (users, orders, and sessions) into a feature matrix. The key challenge here is ensuring that we don't end up with multiple rows per user due to the join operations, which could otherwise inflate our dataset significantly.

#### First-Principles Solution
Identify Common Keys: Each table has columns that can be used as keys to link them together.
Join Conditions: Use appropriate JOIN conditions to ensure we match records correctly.
Inner Join: To avoid excessive rows, use an inner join where the combination of user IDs and other relevant identifiers should be unique for each record.
Optional Filters or Aggregations: Apply filters or aggregations on orders and sessions tables based on time or other criteria to ensure we capture relevant features without duplicating data.
SQL Query Example

Assuming:

users table has columns: user_id, username
orders table has columns: order_id, user_id, order_date, total_amount
sessions table has columns: session_id, user_id, start_time, end_time

Here is the SQL query:

SQL
SELECT
    u.user_id,
    u.username,
    o.order_id AS last_order_id,
    MAX(o.order_date) AS last_order_date,
    MAX(o.total_amount) AS total_last_order_amount,
    s.session_id AS last_session_id,
    MAX(s.start_time) AS start_last_session_time,
    MAX(s.end_time) AS end_last_session_time
FROM
    users u
JOIN
    orders o ON u.user_id = o.user_id
JOIN
    sessions s ON u.user_id = s.user_id
WHERE
    -- Optional: Filter recent or active sessions/orders
    o.order_date >= DATE_SUB(CURRENT_DATE, INTERVAL 30 DAY)
GROUP BY
    u.user_id;

#### Closing Summary
+ Common Mistakes/Follow-Ups

This query ensures that we get a single row per user by using an INNER JOIN and grouping. The key to avoiding inflated rows is through proper filtering and aggregation. Common mistakes include:

Not applying filters, leading to unnecessary rows.
Using incorrect join conditions or types (e.g., outer joins).
Forgetting to aggregate relevant data from joined tables.

#### Follow-Ups
How would you handle users with no orders or sessions?
What if the sessions table has multiple entries per day? Would you want to capture all of them, or just one?
How can you further optimize this query for large datasets?

**Comparison:**

| user_id | username | last_order_id | last_order_date     | total_last_order_amount |
|---------|----------|---------------|---------------------|------------------------|
| 1       | john_doe | 102           | 2023-10-02T17:00    | 49.99                  |
| 2       | jane_doe | 102           | 2023-10-02T18:00    | 49.99                  |

**Code:**

```sql
SELECT
    u.user_id,
    u.username,
    o.order_id AS last_order_id,
    MAX(o.order_date) AS last_order_date,
    MAX(o.total_amount) AS total_last_order_amount,
    s.session_id AS last_session_id,
    MAX(s.start_time) AS start_last_session_time,
    MAX(s.end_time) AS end_last_session_time
FROM
    users u
JOIN
    orders o ON u.user_id = o.user_id
JOIN
    sessions s ON u.user_id = s.user_id
WHERE
    o.order_date >= DATE_SUB(CURRENT_DATE, INTERVAL 30 DAY)
GROUP BY
    u.user_id;
```

**Example input / output:**

```text
-- Sample input and output

Input:
users, orders, sessions with sample data above.

Output:
+---------+----------+-----------+---------------------+-----------------------+
| user_id | username | last_order_id | last_order_date     | total_last_order_amount |
+---------+----------+---------------+---------------------+------------------------+
| 1       | john_doe | 102           | 2023-10-02T17:00    | 49.99                  |
+---------+----------+---------------+---------------------+------------------------+
| 2       | jane_doe | 102           | 2023-10-02T18:00    | 49.99                  |
+---------+----------+---------------+---------------------+------------------------+
```

**Data:**

```sql
-- Sample data schemas

users:
user_id | username
1       | john_doe
2       | jane_doe

orders:
order_id | user_id   | order_date        | total_amount
101      | 1         | 2023-10-01T14:00  | 59.99
102      | 2         | 2023-10-02T17:00  | 49.99

sessions:
session_id | user_id   | start_time        | end_time
1001       | 1         | 2023-10-01T15:00  | 2023-10-01T16:00
1002       | 2         | 2023-10-02T18:00  | 2023-10-02T19:00
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Is user_id the same?
7. Join orders on user_id
8. Apply filters to recent data?
9. Join sessions on user_id
10. Group by user_id
11. Aggregate order and sessiondata
12. End

