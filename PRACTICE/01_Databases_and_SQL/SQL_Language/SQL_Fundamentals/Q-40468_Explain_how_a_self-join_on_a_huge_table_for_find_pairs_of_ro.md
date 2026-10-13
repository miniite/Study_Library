---
type: "practice"
difficulty: "Deep"
---

# Explain how a self-join on a huge table for 'find pairs of rows within 5 minutes of each other' becomes O(n^2), and how a range join with a bounded window plus an index on the timestamp makes it tractable.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Attribution with Window Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Attribution_with_Window_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a huge table to find pairs of rows within 5 minutes of each other, a self-join can initially seem like a straightforward solution. However, it often leads to an (O(n^2)) complexity due to the nature of how such joins are executed. In contrast, using a range join with a bounded window and indexing on the timestamp makes this operation more efficient and scalable.

Let's dive into the details step-by-step:

#### Triaging the Question

The core question is about optimizing a query that finds pairs of rows within 5 minutes of each other from a large table. The key points to consider are:

Self-Join Complexity: Understanding why a naive self-join results in (O(n^2)) complexity.
Range Join with Bounded Window and Indexing: Exploring how this approach can reduce the complexity to something more manageable.

#### Assumptions
The table has a column timestamp representing when each row was inserted or updated.
We want pairs of rows where the difference in their timestamps is within 5 minutes (300 seconds).
The self-join and range join are performed on this large table, which we assume to have (n) rows.
Self-Join Complexity
Naive Self-Join Implementation

A naive self-join involves joining a table with itself. The query might look something like:

SQL
SELECT t1.*, t2.*
FROM my_table t1
JOIN my_table t2
ON t1.timestamp <= t2.timestamp + 300 AND t1.timestamp >= t2.timestamp - 300;

This approach checks every possible pair of rows. For each row (t1), it performs a join with all other rows in the table, leading to:

Row 1 is compared with Rows 2 to n.
Row 2 is compared with Rows 3 to n, and so on.

The total number of comparisons can be calculated as: [ \sum_{i=1}^{n-1} i = \frac{n(n-1)}{2} \approx O(n^2) ]

Range Join with Bounded Window
Optimized Approach

To optimize the query, we leverage a range join along with an index on the timestamp column. The idea is to use an index to efficiently filter rows that are within 5 minutes of each other.

The SQL might look like:

SQL
SELECT t1.*, t2.*
FROM my_table t1
JOIN (SELECT * FROM my_table WHERE timestamp BETWEEN t1.timestamp - 300 AND t1.timestamp + 300) t2
ON true; -- This is a dummy join condition since we only care about the subquery result
Explanation of the Approach
For each row t1, we query for all rows in t2 where the timestamp is between t1.timestamp - 300 and t1.timestamp + 300.
This subquery ensures that only relevant rows are considered, significantly reducing the number of comparisons.
Why It Works
The index on timestamp helps in quickly filtering out rows that do not fall within the 5-minute window.
For each row (t1), we perform a much smaller set of comparisons compared to the full self-join.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, while a naive self-join on a large table can lead to an unmanageable (O(n^2)) complexity, using a range join with a bounded window and indexing on timestamp allows us to efficiently find pairs of rows within 5 minutes of each other.

#### Common mistakes
include:

Not Using Indexes: Failing to take advantage of the index can significantly slow down the query.
Incorrect Window Boundaries: Ensuring that the timestamp difference is accurately bounded by 300 seconds.
Ignoring Data Distribution: The performance can degrade if data is not uniformly distributed over time.

#### Follow-ups
might include discussing further optimizations, such as partitioning large tables or using more advanced database features like window functions in SQL.

**Comparison:**

| id | timestamp            |
|----|----------------------|
| 1  | 2023-10-01T09:00:00Z |
| 2  | 2023-10-01T09:05:30Z |
| 3  | 2023-10-01T09:10:45Z |
| 4  | 2023-10-01T09:15:20Z |

**Code:**

```sql
-- Naive Self-Join
SELECT t1.*, t2.*
FROM my_table t1
JOIN my_table t2
ON t1.timestamp <= t2.timestamp + 300 AND t1.timestamp >= t2.timestamp - 300;

-- Optimized Range Join with Index
SELECT t1.*, t2.*
FROM my_table t1
JOIN (SELECT * FROM my_table WHERE timestamp BETWEEN t1.timestamp - 300 AND t1.timestamp + 300) t2
ON true;
```

**Example input / output:**

```text
-- Input: Find pairs of rows within 5 minutes of each other for the given data

-- Naive Self-Join - Unoptimized Query
SELECT t1.id AS id1, t2.id AS id2, t1.timestamp AS t1_timestamp, t2.timestamp AS t2_timestamp
FROM my_table t1
JOIN my_table t2
ON t1.timestamp <= t2.timestamp + 300 AND t1.timestamp >= t2.timestamp - 300;

-- Output (Example):
+-----+-----+-------------------+-------------------+
| id1 | id2 | t1_timestamp      | t2_timestamp      |
+-----+-----+-------------------+-------------------+
| 1   | 2   | 2023-10-01T09:00:00Z | 2023-10-01T09:05:30Z |
| 1   | 3   | 2023-10-01T09:00:00Z | 2023-10-01T09:10:45Z |
+-----+-----+-------------------+-------------------+

-- Optimized Range Join with Index
SELECT t1.id AS id1, t2.id AS id2, t1.timestamp AS t1_timestamp, t2.timestamp AS t2_timestamp
FROM my_table t1
JOIN (SELECT * FROM my_table WHERE timestamp BETWEEN t1.timestamp - 300 AND t1.timestamp + 300) t2
ON true;

-- Output (Example):
+-----+-----+-------------------+-------------------+
| id1 | id2 | t1_timestamp      | t2_timestamp      |
+-----+-----+-------------------+-------------------+
| 1   | 2   | 2023-10-01T09:00:00Z | 2023-10-01T09:05:30Z |
+-----+-----+-------------------+-------------------+
```

**Data:**

```json
{
  "my_table": {
    "columns": ["id", "timestamp"],
    "data": [
      {"id": 1, "timestamp": "2023-10-01T09:00:00Z"},
      {"id": 2, "timestamp": "2023-10-01T09:05:30Z"},
      {"id": 3, "timestamp": "2023-10-01T09:10:45Z"},
      {"id": 4, "timestamp": "2023-10-01T09:15:20Z"}
    ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is the table large?
5. Use Range Join with Index
6. Use Naive Self-Join
7. Find pairs within 5 minutesefficiently
8. Perform full self-join, O(n^2)

