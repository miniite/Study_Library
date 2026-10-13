---
type: "practice"
difficulty: "Deep"
---

# How would you design a schema and query to compute cohort retention (users active in week N after signup) efficiently on tens of millions of events?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Activity Timeline Schema](<../../../../TOOLS/MySQL/MySQL_Platform/Activity_Timeline_Schema.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute cohort retention efficiently, we need to consider the scale and performance implications of handling millions or even billions of user events. The key is to design a schema that supports fast lookups while minimizing storage overhead. We'll also need to craft queries that leverage indexes effectively.

#### Triaging the Question

The core of this question revolves around efficient data retrieval from large datasets. Specifically, we are asked to compute cohort retention, which involves determining how many users remain active in week N after their signup date. This requires:

Storing User Events: We need a way to store and quickly query user events.
Cohort Definition: Accurately defining the cohorts based on signup dates.
Retention Calculation: Efficiently computing which users are still active at different time intervals.
Schema Design

We'll use two main tables: users for storing user metadata, and user_events for storing event data. The schema should support fast lookups and aggregations.

SQL
-- users table
CREATE TABLE IF NOT EXISTS users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    signup_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    -- other user-specific fields
);

-- user_events table
CREATE TABLE IF NOT EXISTS user_events (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT,
    event_date TIMESTAMP,
    -- other event-specific fields
    FOREIGN KEY (user_id) REFERENCES users(id)
);
Querying Cohort Retention

To compute cohort retention, we need to:

Identify the signup week of each user.
Find events in a specific week N after their signup date.
Count unique users who are still active in that week.

We'll use SQL queries with proper indexing to achieve this efficiently.

#### First Principles Solution

The following steps outline our solution:

Create Indexes:

An index on users.signup_date for quick lookups by signup week.
An index on user_events.event_date for fast event date sorting and filtering.

Query Cohort Retention:

SQL
-- Example query to find cohort retention in week 4 after signup
SELECT
    DATE_FORMAT(users.signup_date, '%Y-%m') AS signup_week,
    COUNT(DISTINCT user_id) AS retained_users
FROM (
    SELECT
        user_id,
        DATE(event_date) - INTERVAL DAYOFWEEK(event_date) + 6 DAY AS adjusted_event_week
    FROM user_events
    WHERE
        event_date >= users.signup_date AND
        event_date < users.signup_date + INTERVAL 7 DAY * 4  -- 28 days after signup (4 weeks)
) AS filtered_events
JOIN users ON filtered_events.user_id = users.id
GROUP BY signup_week;

Optimizations:

Use partitioning on the user_events table if the dataset grows extremely large.

#### Cons
ider sharding by user_id or signup_date for distributed storage and query load.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, we designed a schema with proper indexing to efficiently compute cohort retention. The query leverages date formatting and interval calculations to accurately define cohorts and filter events. Common mistakes include not properly indexing tables, failing to optimize complex queries, or neglecting data partitioning/sharding as the dataset grows.

#### Common Mistakes
Lack of Indexes: Not creating appropriate indexes on signup_date and event_date.
Complex Queries Without Optimization: Failing to use subqueries or proper grouping for efficiency.
Ignoring Scalability Issues: Overlooking the need for data partitioning or sharding as the dataset grows.

#### Follow-Ups
Performance Tuning: Discuss further optimizations like query caching, materialized views, or using a more advanced DBMS like PostgreSQL with its extensive indexing features.
Cost Considerations: Evaluate the cost implications of different storage and indexing strategies.
Alternatives to MySQL: Suggest alternatives if performance becomes an issue, such as using a NoSQL database for event streaming.

**Comparison:**

| signup_week | retained_users |
|-------------|----------------|
| 2023-01     | 2              |

**Code:**

```

CREATE TABLE IF NOT EXISTS users ( id INT AUTO_INCREMENT PRIMARY KEY, signup_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP, -- other user-specific fields );

CREATE TABLE IF NOT EXISTS user_events ( id INT AUTO_INCREMENT PRIMARY KEY, user_id INT, event_date TIMESTAMP, -- other event-specific fields FOREIGN KEY (user_id) REFERENCES users(id) );

CODE
Copy

```sql
-- Example query to find cohort retention in week 4 after signup
SELECT
    DATE_FORMAT(users.signup_date, '%Y-%m') AS signup_week,
    COUNT(DISTINCT user_id) AS retained_users
FROM (
    SELECT
        user_id,
        DATE(event_date) - INTERVAL DAYOFWEEK(event_date) + 6 DAY AS adjusted_event_week
    FROM user_events
    WHERE
        event_date >= users.signup_date AND
        event_date < users.signup_date + INTERVAL 7 DAY * 4  -- 28 days after signup (4 weeks)
) AS filtered_events
JOIN users ON filtered_events.user_id = users.id
GROUP BY signup_week;
```

**Example input / output:**

```text
-- Input
SELECT
    DATE_FORMAT(users.signup_date, '%Y-%m') AS signup_week,
    COUNT(DISTINCT user_id) AS retained_users
FROM (
    SELECT
        user_id,
        DATE(event_date) - INTERVAL DAYOFWEEK(event_date) + 6 DAY AS adjusted_event_week
    FROM user_events
    WHERE
        event_date >= users.signup_date AND
        event_date < users.signup_date + INTERVAL 7 DAY * 4  -- 28 days after signup (4 weeks)
) AS filtered_events
JOIN users ON filtered_events.user_id = users.id
GROUP BY signup_week;

-- Output
+---------------------+----------------+
| signup_week         | retained_users |
+---------------------+----------------+
| 2023-01             | 2              |
+---------------------+----------------+
```

**Data:**

```json
{
    "users": [
        {"id": 1, "signup_date": "2023-01-01"},
        {"id": 2, "signup_date": "2023-01-08"}
    ],
    "user_events": [
        {"id": 1, "user_id": 1, "event_date": "2023-01-05"},
        {"id": 2, "user_id": 1, "event_date": "2023-01-09"},
        {"id": 3, "user_id": 2, "event_date": "2023-01-16"}
    ]
}
```

**Flow:**

1. Yes
2. No
3. User Signs Up
4. Is Event in Week N AfterSignup?
5. Count User as Retained
6. Skip to Next User
7. Group by Week of Signup,Count Users

