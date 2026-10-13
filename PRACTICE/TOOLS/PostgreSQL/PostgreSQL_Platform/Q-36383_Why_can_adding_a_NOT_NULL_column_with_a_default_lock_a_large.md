---
type: "practice"
difficulty: "Expert"
---

# Why can adding a NOT NULL column with a default lock a large table, and how do you avoid it?

**Difficulty:** Expert · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Safe Migrations on Rolling Deploys](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Safe_Migrations_on_Rolling_Deploys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why adding a NOT NULL column with a default might lock up a large PostgreSQL table, we need to delve into how PostgreSQL handles such operations under the hood. This operation involves changing the schema of an existing table and inserting new data, which can be resource-intensive. Let's break down the core issue, explore the process step-by-step, and discuss strategies to avoid it.

#### Triaging the Question

The question is essentially asking why adding a NOT NULL column with a default value might result in a significant lock on a large PostgreSQL table, and how one could prevent such a situation. This involves understanding the internal mechanisms of PostgreSQL during schema changes, particularly around ALTER TABLE operations.

#### Assumptions
The table has many rows (i.e., it is large).
Adding a NOT NULL column with a default value means that every existing row must have a valid value for this new column.
PostgreSQL uses locks to ensure data integrity and consistency during schema changes.

#### First-Principles Solution
Step 1: Understanding the ALTER TABLE Operation

When you execute an ALTER TABLE ... ADD COLUMN NOT NULL DEFAULT ..., PostgreSQL needs to:

Add a new column to the table's metadata.
Ensure that every existing row in the table has a value for this new column.

The second step is critical because, by default, adding a NOT NULL column without specifying a default value requires each row to have data immediately after insertion or modification. If no value is provided during such operations, PostgreSQL will raise an error.

Step 2: Locking Mechanism

PostgreSQL employs various types of locks, including shared and exclusive locks, to manage concurrency and ensure that modifications do not interfere with other transactions. When adding a NOT NULL column, the system needs to acquire an exclusive lock on each row in the table during the operation.

Step 3: Implications for Large Tables

For a large table, this means every row must be locked one by one while the ALTER TABLE command processes. This can lead to significant contention and performance issues:

Lock Contention: All rows are effectively blocked until the operation is complete.
Durability Concerns: The system might need to maintain these locks for an extended period, leading to potential deadlocks if other transactions try to access the table during this time.
Avoiding Lock Contention

To mitigate these issues, several strategies can be employed:

1. Using a Temporary Column with Default Value

One approach is to:

Add the column as NOT NULL but provide a default value that doesn’t impose constraints immediately.
Use UPDATE statements to populate the new column after ensuring all rows are updated.

#### Example

SQL
-- Step 1: Add the column temporarily without constraints
ALTER TABLE my_table ADD COLUMN new_column INT DEFAULT -999; -- Use a sentinel value

-- Step 2: Populate the column with data
UPDATE my_table SET new_column = 42;

-- Step 3: Make the column NOT NULL and remove default
ALTER TABLE my_table ALTER COLUMN new_column SET NOT NULL;
2. Using WITH NO KEY UPDATE

Another method is to use the NO KEY UPDATE option during the ALTER TABLE operation:

SQL
ALTER TABLE my_table ADD COLUMN new_column INT DEFAULT -999 WITH NO KEY UPDATE; -- Temporarily prevent updates

This can reduce lock contention as it allows the system to handle row-level locks more efficiently.

3. Partitioning and Staged Updates

For extremely large tables, partitioning might be necessary:

Split the table into smaller partitions.
Perform ALTER TABLE on each partition separately or in a controlled manner.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Adding a NOT NULL column with a default can lock up large PostgreSQL tables due to the need to update every row. To avoid this, use strategies like temporary columns, staged updates, and careful schema design. Always consider the size of your table and the potential impact on performance and concurrency.

#### Common mistakes
include failing to manage locks properly or not understanding the implications of ALTER TABLE operations on large datasets.

For follow-ups:

Discuss how database versioning (like Postgres 14+ with pg_dump) can help in managing such operations.
Explore alternative methods like using materialized views for reporting purposes, which can reduce direct table access during schema changes.

