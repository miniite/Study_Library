---
type: "practice"
difficulty: "Deep"
---

# You retire old time-series data monthly; explain why DETACH PARTITION plus DROP is dramatically better than a huge DELETE, and what FK and index edge cases complicate detaching a partition.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Retention Purging at Scale](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Retention_Purging_at_Scale.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In PostgreSQL, retiring old time-series data can be optimized significantly by using DETACH PARTITION followed by DROP. This approach is dramatically better than a massive DELETE operation because it minimizes lock contention and reduces the impact on other database operations. However, this method introduces complexities around foreign keys (FKs) and indexes that must be carefully managed.

Let's start with why using DETACH PARTITION followed by DROP is superior to DELETE.

Lock Contention: When you delete a large volume of data with DELETE, PostgreSQL locks the rows being deleted, which can block other transactions from accessing or modifying those rows. This locking mechanism can degrade performance and affect the overall availability of your database.
Partition Management: By using partitioning, you logically divide large tables into smaller chunks that are easier to manage and optimize. Detaching a partition essentially moves data out of the main table without affecting other partitions.

Now let's dive deeper into why FKs and indexes complicate detaching a partition:

Foreign Keys (FKs): If your time-series data is related through foreign keys, you need to ensure that no referential integrity violations occur when moving or removing data. For example, if orders table has a foreign key constraint referencing the customers table, and you are detaching partitions from the orders table, you must guarantee that all referenced customers still exist in their respective partitions.
Indexes: Partitioning often involves creating indexes on each partitioned column to improve query performance. When you detach a partition, you need to ensure that the necessary indexes are updated or maintained to reflect the new state of the database.

#### First-Principles Solution

Design Considerations:

Ensure that your schema supports partitioning well.
Define proper foreign key constraints and manage them carefully during partition operations.
Plan for index management, ensuring that indexes remain valid after detaching partitions.

Step-by-Step Process:

Create Partitions: Use CREATE TABLE with a PARTITION BY clause to define your initial partitions.
SQL
CREATE TABLE sales (
    id SERIAL PRIMARY KEY,
    sale_date DATE NOT NULL,
    amount DECIMAL(10, 2) NOT NULL
) PARTITION BY RANGE (sale_date);
Insert Data: Insert data into the appropriate partitioned tables based on the date range.
SQL
INSERT INTO sales_202301 VALUES (...), (...); -- For January 2023

Detach Partitions:

To detach a partition, use ALTER TABLE. This moves data out of the main table but does not delete it.
SQL
ALTER TABLE sales DETACH PARTITION sales_202301;

Drop Partitions:

Once detached and no longer needed, you can safely drop the partitioned table using DROP TABLE.
SQL
DROP TABLE sales_202301;

#### Common Mistakes
and Follow-Ups
Forgetting FK Constraints: Failing to update or remove foreign key constraints when detaching partitions can lead to integrity issues.
Index Management: Not properly managing indexes during partition operations can result in performance degradation.
Data Consistency: Ensuring that data is consistent across all partitions, especially if there are overlapping ranges.

#### Closing Summary

Using DETACH PARTITION followed by DROP for retiring old time-series data is a more efficient and scalable approach compared to bulk deletions. However, it requires careful management of foreign keys and indexes to avoid potential issues such as integrity violations or performance bottlenecks.

**Comparison:**

| id | sale_date  | amount |
|----|------------|--------|
| 1  | 2023-01-01 | 50.99  |
| 2  | 2023-01-02 | 78.45  |

**Code:**

```sql
-- Example of creating partitions
CREATE TABLE sales (
    id SERIAL PRIMARY KEY,
    sale_date DATE NOT NULL,
    amount DECIMAL(10, 2) NOT NULL
) PARTITION BY RANGE (sale_date);

-- Inserting data into a specific partition
INSERT INTO sales_202301 VALUES (...), (...);

-- Detaching and dropping the partitioned table
ALTER TABLE sales DETACH PARTITION sales_202301;
DROP TABLE sales_202301;
```

**Example input / output:**

```text
-- Example of a DELETE operation
DELETE FROM sales WHERE sale_date < '2023-01-01';
-- Example of DETACH PARTITION followed by DROP
ALTER TABLE sales DETACH PARTITION sales_202301;
DROP TABLE sales_202301;
```

**Data:**

```json
{
  "sales": {
    "id": [1, 2, 3],
    "sale_date": ["2023-01-01", "2023-01-02", "2023-02-01"],
    "amount": [50.99, 78.45, 100.00]
  }
}
```

**Flow:**

1. Create Partitions
2. Insert Data
3. Detaching Partitions
4. Drop Partitions
5. Update FK and Indexes ifNeeded

