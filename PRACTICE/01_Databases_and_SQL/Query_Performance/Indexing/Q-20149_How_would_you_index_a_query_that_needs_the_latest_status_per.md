---
type: "practice"
difficulty: "Deep"
---

# How would you index a query that needs the latest status per entity across millions of entities, comparing DISTINCT ON, a lateral join, and a current-state table?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Join Algorithm Selection](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Join_Algorithm_Selection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of indexing a query to return the latest status per entity from millions of records, we need to consider three approaches: using DISTINCT ON, leveraging a lateral join, and utilizing a current-state table. Each method has its own trade-offs in terms of performance, complexity, and maintenance.

#### Triaging the Question

The core question is about choosing the most efficient way to retrieve the latest status for each entity. The interviewer is probing your understanding of database indexing strategies, query optimization techniques, and the trade-offs between different approaches. We will make several key assumptions:

The entities are identified by a unique identifier (ID).
Status changes are timestamped.
Performance and scalability are critical concerns.
Step-by-Step Solution from First Principles
1. Understanding DISTINCT ON

DISTINCT ON is a PostgreSQL-specific feature that returns the first row of each group defined by one or more columns. To get the latest status, we can use an ORDER BY clause with the timestamp column and then apply DISTINCT ON.

SQL
SELECT DISTINCT ON (entity_id) entity_id, status, timestamp
FROM statuses
ORDER BY entity_id, timestamp DESC;

#### Pros

Simple to implement.
Efficient for small-to-medium datasets.

#### Cons

Performance degrades as the dataset grows due to sorting and DISTINCT operations.
Not optimal for large-scale databases.
2. Lateral Join

A lateral join involves a subquery that is evaluated once per row of the outer query, allowing us to reference columns from the outer query in the subquery. This can be more efficient than DISTINCT ON for complex queries.

SQL
SELECT s.entity_id, s.status, s.timestamp
FROM statuses s
JOIN (
    SELECT entity_id, MAX(timestamp) AS max_timestamp
    FROM statuses
    GROUP BY entity_id
) latest ON s.entity_id = latest.entity_id AND s.timestamp = latest.max_timestamp;

#### Pros

Can handle more complex queries.
Generally more performant than DISTINCT ON for larger datasets.

#### Cons

More complex query structure.
Requires additional join operations, which can be costly.
3. Current-State Table

Maintaining a current-state table that holds the latest status for each entity reduces the need for frequent queries on the statuses table. This approach involves periodic updates to the current-state table using triggers or scheduled tasks.

Current-State Schema:

SQL
CREATE TABLE current_state (
    entity_id INT PRIMARY KEY,
    status TEXT NOT NULL,
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
Implementing a Current-State Table

Trigger for Real-Time Updates:

SQL
CREATE OR REPLACE FUNCTION update_current_state()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO current_state (entity_id, status) VALUES (NEW.entity_id, NEW.status)
    ON CONFLICT (entity_id) DO UPDATE SET status = EXCLUDED.status, last_updated = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_current_state_trigger
AFTER INSERT OR UPDATE ON statuses
FOR EACH ROW EXECUTE FUNCTION update_current_state();

Query for Latest Status:

SQL
SELECT entity_id, status FROM current_state;

#### Pros

Highly performant as it avoids expensive join operations.
Ensures consistency and up-to-date data.

#### Cons

Requires additional storage and maintenance overhead.
May have latency between updates and the status being reflected in the table.

#### Closing Summary

Each method—DISTINCT ON, lateral join, and current-state table—has its strengths and weaknesses. For a large-scale database with millions of entities, the current-state approach is likely the most efficient due to reduced query complexity and consistent data availability. However, it requires ongoing maintenance and additional storage.

#### Common Mistakes/Follow-Ups
Ignoring Indexing: Ensure that appropriate indexes are in place for columns used in ORDER BY, JOIN, and primary keys.
Overlooking Performance Monitoring: Regularly monitor query performance and adjust indexing or schema as needed.
Ignoring Data Consistency: Ensure that the current-state table is updated atomically to avoid data inconsistencies.

**Comparison:**

| Entity ID | Status    | Timestamp           |
| --------- | --------- | ------------------- |
| 1         | Inactive  | 2023-10-01 10:00:00 |
| 2         | Active    | 2023-10-02 11:00:00 |

**Code:**

```sql
-- DISTINCT ON Example
SELECT DISTINCT ON (entity_id) entity_id, status, timestamp
FROM statuses
ORDER BY entity_id, timestamp DESC;

-- Lateral Join Example
SELECT s.entity_id, s.status, s.timestamp
FROM statuses s
JOIN (
    SELECT entity_id, MAX(timestamp) AS max_timestamp
    FROM statuses
    GROUP BY entity_id
) latest ON s.entity_id = latest.entity_id AND s.timestamp = latest.max_timestamp;

-- Current-State Table Schema and Trigger
CREATE TABLE current_state (
    entity_id INT PRIMARY KEY,
    status TEXT NOT NULL,
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE OR REPLACE FUNCTION update_current_state()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO current_state (entity_id, status) VALUES (NEW.entity_id, NEW.status)
    ON CONFLICT (entity_id) DO UPDATE SET status = EXCLUDED.status, last_updated = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_current_state_trigger
AFTER INSERT OR UPDATE ON statuses
FOR EACH ROW EXECUTE FUNCTION update_current_state();
```

**Example input / output:**

```text
-- Using DISTINCT ON
SELECT DISTINCT ON (entity_id) entity_id, status, timestamp
FROM statuses
ORDER BY entity_id, timestamp DESC;

-- Expected Output:
+---------+----------+---------------------+
| entity_id | status   | timestamp           |
+---------+----------+---------------------+
| 1        | Inactive | 2023-10-01 10:00:00 |
| 2        | Active   | 2023-10-02 11:00:00 |
+---------+----------+---------------------+

-- Using Lateral Join
SELECT s.entity_id, s.status, s.timestamp
FROM statuses s
JOIN (
    SELECT entity_id, MAX(timestamp) AS max_timestamp
    FROM statuses
    GROUP BY entity_id
) latest ON s.entity_id = latest.entity_id AND s.timestamp = latest.max_timestamp;

-- Expected Output:
+---------+----------+---------------------+
| entity_id | status   | timestamp           |
+---------+----------+---------------------+
| 1        | Inactive | 2023-10-01 10:00:00 |
| 2        | Active   | 2023-10-02 11:00:00 |
+---------+----------+---------------------+

-- Using Current-State Table
SELECT entity_id, status FROM current_state;

-- Expected Output:
+---------+----------+
| entity_id | status   |
+---------+----------+
| 1        | Inactive |
| 2        | Active   |
+---------+----------+
```

**Data:**

```sql
-- Sample Data for Statuses Table
INSERT INTO statuses (entity_id, status, timestamp) VALUES
(1, 'Active', '2023-10-01 09:00:00'),
(1, 'Inactive', '2023-10-01 10:00:00'),
(2, 'Active', '2023-10-02 11:00:00');

-- Sample Current-State Table Data
INSERT INTO current_state (entity_id, status) VALUES
(1, 'Inactive'),
(2, 'Active');
```

**Flow:**

1. Current-State Table
2. Select latest status
3. Statuses Table
4. Order by timestamp
5. DISTINCT ON
6. Statuses Table
7. Min timestamp per entity
8. Lateral Join

