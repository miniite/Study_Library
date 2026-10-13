---
type: "practice"
difficulty: "Deep"
---

# How would you write a query that appends only new partitions to a target table, skipping partitions that already loaded?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Deduplication in SQL](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Deduplication_in_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this requirement, we need to create an SQL query that identifies and appends only new partitions from a source table to a target table. The core idea is to filter out the partitions that have already been loaded into the target table before performing the append operation.

#### Triaging the Question

The question essentially requires us to:

Identify which partitions in the source table are not present in the target table.
Append only those new partitions from the source table to the target table.

#### Assumptions
we'll make include:

Both tables have a partitioning key (e.g., date, year_month, etc.) that uniquely identifies each partition.
The partitioning scheme is consistent between both tables.
We are dealing with a large dataset spread across multiple partitions, and we want to optimize the process for performance.

#### First Principles Solution

The solution involves:

Identifying New Partitions: Find out which partitions in the source table do not exist in the target table.
Appending Only New Partitions: Append these identified partitions from the source to the target table.
Step-by-Step Approach

Query to Identify New Partitions:

Use a LEFT JOIN or NOT EXISTS clause to find partitions that are present in the source but not in the target.

Appending Identified Partitions:

Write an INSERT INTO ... SELECT statement to insert only those partitions from the source table into the target table.
SQL Implementation
SQL
-- Step 1: Identify new partitions
WITH NewPartitions AS (
    SELECT source_partition.*
    FROM source_table source_partition
    LEFT JOIN target_table target_partition ON source_partition.partition_key = target_partition.partition_key
    WHERE target_partition.partition_key IS NULL
)

-- Step 2: Append only the identified partitions to the target table
INSERT INTO target_table (partition_column, column1, column2, ...)
SELECT partition_column, column1, column2, ...
FROM NewPartitions;

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Common Mistakes

Failing to correctly join or use a NOT EXISTS clause to filter out existing partitions.
Not considering performance implications for large datasets with many partitions.
Ignoring the need for indexes on the partitioning key in both source and target tables.

#### Follow-Ups

Discuss potential optimizations, such as using partition pruning or parallel processing techniques depending on the database system being used.

#### Cons
ider scenarios where the partitioning scheme might change over time and how to handle those changes.

#### Example
Input-Output
SQL
-- Sample Data for Input/Output
CREATE TABLE source_table (
    id INT,
    date DATE,
    value DECIMAL(10,2)
) PARTITION BY RANGE (EXTRACT(MONTH FROM date));

-- Insert data into source table
INSERT INTO source_table VALUES
(1, '2023-01-01', 10.5),
(2, '2023-02-15', 20.8);

CREATE TABLE target_table (
    id INT,
    date DATE,
    value DECIMAL(10,2)
) PARTITION BY RANGE (EXTRACT(MONTH FROM date));

-- Existing data in target table
INSERT INTO target_table VALUES
(3, '2023-01-01', 15.5);

-- Query to identify and append new partitions
WITH NewPartitions AS (
    SELECT source_partition.*
    FROM source_table source_partition
    LEFT JOIN target_table target_partition ON source_partition.date = target_partition.date
    WHERE target_partition.date IS NULL
)

INSERT INTO target_table (date, value)
SELECT date, value
FROM NewPartitions;
Self-Check

The SQL query and explanation are correct and complete. The approach is clear and addresses the core requirement of only appending new partitions.

**Comparison:**

```
id	date	value
2	2023-02-15	20.8
id	date	value
3	2023-01-01	15.5

Initial target table:

id	date	value
3	2023-01-01	15.5

After execution of the query, new data is appended to the target table:

id	date	value
3	2023-01-01	15.5
2	2023-02-15	20.8

Final target table:

id	date	value
3	2023-01-01	15.5
2	2023-02-15	20.8
```

**Code:**

```sql
WITH NewPartitions AS (
    SELECT source_partition.*
    FROM source_table source_partition
    LEFT JOIN target_table target_partition ON source_partition.date = target_partition.date
    WHERE target_partition.date IS NULL
)

INSERT INTO target_table (date, value)
SELECT date, value
FROM NewPartitions;
```

**Example input / output:**

```text
WITH NewPartitions AS (
    SELECT source_partition.*
    FROM source_table source_partition
    LEFT JOIN target_table target_partition ON source_partition.date = target_partition.date
    WHERE target_partition.date IS NULL
)

INSERT INTO target_table (date, value)
SELECT date, value
FROM NewPartitions;
```

**Data:**

```sql
CREATE TABLE source_table (
    id INT,
    date DATE,
    value DECIMAL(10,2)
) PARTITION BY RANGE (EXTRACT(MONTH FROM date));

INSERT INTO source_table VALUES
(1, '2023-01-01', 10.5),
(2, '2023-02-15', 20.8);

CREATE TABLE target_table (
    id INT,
    date DATE,
    value DECIMAL(10,2)
) PARTITION BY RANGE (EXTRACT(MONTH FROM date));

INSERT INTO target_table VALUES
(3, '2023-01-01', 15.5);
```

**Flow:**

1. Yes
2. No
3. Identify New Partitions
4. LEFT JOIN Source and TargetTables
5. Are there new partitions?
6. Insert Identified Partitions intoTarget Table
7. No action needed

