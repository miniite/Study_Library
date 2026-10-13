---
type: "practice"
difficulty: "Deep"
---

# How do you drop old data cheaply with partition detach/drop instead of a giant DELETE, and why is that so much lighter on the database?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Batching Writes](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Batching_Writes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with large volumes of historical or outdated data in a database management system (DBMS), using partitioning can be a more efficient and cost-effective approach compared to performing massive DELETE operations. Specifically, you can use partition detach/drop techniques to remove old data without locking up the entire table. This method is particularly beneficial for read-heavy systems where downtime or performance degradation due to heavy transactions is undesirable.

The core idea here is that by organizing data into partitions and then dropping those partitions, we minimize the impact on database operations and reduce resource consumption. Let's break this down step-by-step from first principles:

Partitioning Data: First, you need to partition your large table based on a relevant criteria such as date or ID ranges. For example, if you have a sales table with historical sales data spanning many years, you might create partitions for each year.
Detaching/ Dropping Partitions: Once the data is partitioned, when it's time to remove old records (e.g., records older than one year), you can detach and drop the corresponding partition(s) instead of deleting them row by row from the main table.

#### Triage
What We're Probing

The interviewer is likely probing your understanding of:

Partitioning Strategies: How partitions are created, managed, and used.

#### Performance Considerations
The efficiency gains achieved through partitioning vs. full table scans or deletes.
Transactional Integrity: Ensuring that dropping a partition does not violate database constraints or cause other issues.

#### First Principles Solution

Data Partitioning:

Create Partitions Based on Time or ID Ranges: Use a date range for time-series data, or use an ID range if the data is less time-sensitive.
SQL
CREATE TABLE sales (
    id INT,
    sale_date DATE,
    amount DECIMAL(10, 2)
) PARTITION BY RANGE (YEAR(sale_date));
Define Partitions for Each Time Range: For example, create partitions sales_2023, sales_2024, etc.

Dropping Old Partitions:

Once the data in a partition becomes old and no longer needed, you can drop that partition without affecting other parts of the table.
SQL
ALTER TABLE sales DROP PARTITION sales_2018;

Detaching Partitions (Optional):

In some DBMS like Oracle, partitions can be detached to reclaim storage space more efficiently.

#### Closing Summary
and Common Mistakes

Using partition detach/drop is a powerful technique for maintaining large historical databases with minimal impact on performance and transactional integrity. However, it requires careful planning and management:

Common Mistake 1: Not properly designing the partitioning strategy can lead to suboptimal performance or increased maintenance overhead.
Common Mistake 2: Failing to consider the implications of partition drops on data dependencies, especially in complex schemas.
Further Considerations
Concurrency Control: Ensure that the operation does not affect concurrent transactions.
Backup and Recovery: Plan for backups before dropping partitions, as this action is irreversible.
Database Schema Design: Properly plan schema design to minimize future maintenance costs associated with partitioning.

**Code:**

```sql
-- Example of creating a partitioned table in PostgreSQL
CREATE TABLE sales (
    id INT,
    sale_date DATE,
    amount DECIMAL(10, 2)
) PARTITION BY RANGE (YEAR(sale_date));

-- Creating partitions for specific years
CREATE TABLE sales_2018 PARTITION OF sales FOR VALUES FROM ('2018-01-01') TO ('2019-01-01');
CREATE TABLE sales_2019 PARTITION OF sales FOR VALUES FROM ('2019-01-01') TO ('2020-01-01');

-- Dropping a partition
ALTER TABLE sales DROP PARTITION sales_2018;
```

