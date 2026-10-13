---
type: "practice"
difficulty: "Deep"
---

# Why is a mass DELETE of millions of rows expensive in Oracle and what alternatives (CTAS, partition drop) do you consider?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Oracle SQL and Indexes](<../../../../TOOLS/Oracle/Oracle_Platform/Oracle_SQL_and_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're dealing with an Oracle database, specifically focusing on high-volume DELETE operations that can be resource-intensive. The main challenge is understanding why large-scale deletions are costly and exploring alternative strategies like CTAS (Create Table As Select) and partition drop to mitigate these costs.

#### Triage
Restating the Question

The question primarily aims at gaining a deep understanding of:

Resource Consumption During Large Deletes: Why does deleting millions of rows consume so much resources?
Alternatives for Efficient Data Deletion: How can we perform data cleanup more efficiently without relying on high-volume deletes?

#### First-Principles Solution
1. Resource Consumption During Large Deletes

When you delete a large number of rows from an Oracle table, several operations are triggered:

Row Locking and Unpinning: Each row being deleted is locked to prevent other transactions from modifying it. Once the delete operation is committed, these locks are released.
Data Block Management: Oracle manages data blocks by marking them as free space. This involves updating the segment header and freelists.
Redo Log Buffer Utilization: Deleting rows generates redo logs, which need to be written to the redo log buffer and then flushed to disk.
2. Why Is It Expensive?

The cost of large deletes can be broken down into several factors:

Lock Contention: Locks on deleted rows prevent other transactions from reading or modifying them.
Disk I/O Operations: Frequent updates to block headers and freelists, along with the generation and flushing of redo logs, increase disk I/O operations.
Memory Usage: The redo log buffer and other internal structures consume significant memory.
3. Alternatives: CTAS (Create Table As Select)

CTAS is an efficient method for copying data from one table to another while filtering out unwanted rows:

SQL
CREATE TABLE new_table AS
SELECT * FROM old_table WHERE condition;
Advantages: This approach avoids the overhead of DELETE operations by only copying desired rows. It's much faster and more resource-friendly.

#### Cons
iderations: Ensure that the schema of the new table matches the old one, and handle any indexes or constraints appropriately.
4. Partition Drop

Partitioning allows you to manage data more granularly:

Create Partitions: Split the large table into smaller, manageable partitions based on a column.
Drop Unwanted Partitions: Once partitioning is in place, you can drop specific partitions containing unwanted rows.
SQL
ALTER TABLE old_table DROP PARTITION p_unwanted;
Advantages: This method minimizes the impact of deletions by reducing the scope to smaller parts of the table.

#### Cons
iderations: Ensure that the dropped partition does not contain critical data, and re-balance or merge partitions as needed.

#### Closing Summary

In summary, large DELETE operations in Oracle are expensive due to extensive lock management, disk I/O, and memory usage. Alternatives like CTAS and partition drop offer more efficient ways to manage data cleanup by reducing resource contention and minimizing the impact of deletions.

#### Common Mistakes/Follow-Ups
Not Considering Indexes: Deleting rows can invalidate indexes, leading to additional overhead.
Ignoring Partitioning: Failing to utilize partitioning can result in unnecessary full table scans during large deletes.
Neglecting Constraints and Triggers: Ensure that constraints and triggers do not interfere with your delete or alternative operations.

**Comparison:**

| old_table | new_table |
|-----------|-----------|
| id        |           |
| name      |           |
| age       |           |
| 1         | 2         |
| 2 (Filtered) | -        |
| 3         | 3         |

**Code:**

```sql
-- Example CTAS Operation
CREATE TABLE new_table AS
SELECT * FROM old_table WHERE condition;
```

**Example input / output:**

```text
Input:
CREATE TABLE new_table AS
SELECT * FROM old_table WHERE age > 30;

Output:
Table new_table created with following rows:
- id: 2, name: Jane Smith, age: 25 (Filtered out)
- id: 3, name: Mike Johnson, age: 40
```

**Data:**

```yaml
old_table:
  - id: 1
    name: John Doe
    age: 30
  - id: 2
    name: Jane Smith
    age: 25
  - id: 3
    name: Mike Johnson
    age: 40
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Is table partitioned?
7. Create new table using CTAS
8. Are specific partitions to bedropped?
9. Drop unwanted partitions
10. Perform large DELETEoperation
11. Rebalance or merge partitionsas needed
12. End

