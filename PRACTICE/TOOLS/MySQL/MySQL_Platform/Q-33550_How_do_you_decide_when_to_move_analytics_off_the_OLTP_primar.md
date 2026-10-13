---
type: "practice"
difficulty: "Deep"
---

# How do you decide when to move analytics off the OLTP primary onto a replica or warehouse, and what query patterns are the signal to do so?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Read Your Writes in MySQL](<../../../../TOOLS/MySQL/MySQL_Platform/Read_Your_Writes_in_MySQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding whether to move analytics queries from an OLTP (Online Transaction Processing) database onto a replica or a separate warehouse, we must balance factors such as performance, cost, and data freshness requirements. The key signals often come from observing query patterns and understanding the characteristics of both types of databases.

Firstly, let's understand what these databases are good at:

OLTP Database: Designed for transaction processing, it ensures ACID properties (Atomicity, Consistency, Isolation, Durability) and low-latency transactions. It is optimized for write-heavy operations.
Replica or Warehouse: Optimized for read-heavy operations with no need to maintain consistency across all writes, making them suitable for complex analytics and reporting.

We typically look at the following query patterns as signals:

Frequency of Query Execution: Frequent queries that run every few minutes are more likely candidates for moving off.
Query Complexity: Queries involving joins, aggregations, or complex subqueries often require more computational resources than simple CRUD operations.
Data Size and Volume: Large datasets processed in batch operations can be better handled by a warehouse system.

The decision process involves:

Identifying High-Volume Read Queries: Use monitoring tools to identify which queries are executed frequently and consume significant CPU or I/O resources on the primary database.
Assessing Query Complexity: Evaluate whether these complex queries can be efficiently offloaded to a replica or warehouse with less stringent consistency requirements.
Testing Replication Strategies: Implement a test setup where some analytics queries run against a secondary replica, measure performance improvements and ensure data freshness is acceptable.

If moving the queries results in significant performance gains (e.g., reduced latency) without compromising critical transactions on the primary database, then it’s likely worth implementing.

**Code:**

```sql
-- Example of a query that could benefit from being moved to a replica or warehouse
SELECT customer_id, SUM(order_amount) AS total_spent
FROM orders
GROUP BY customer_id
ORDER BY total_spent DESC;
```

**Example input / output:**

```

Sample input query:

SQL
Copy
SELECT product_id, COUNT(*) as item_count, AVG(price) as avg_price
FROM sales_data
WHERE date >= '2023-10-01' AND date <= '2023-10-31'
GROUP BY product_id;

Expected output (simplified representation):

PLAINTEXT
Copy
+------------+-------------+---------------+
| product_id | item_count  | avg_price     |
+------------+-------------+---------------+
| 1          | 500         | 79.84         |
| 2          | 300         | 65.23         |
| ...        | ...         | ...           |
+------------+-------------+---------------+
```

**Flow:**

1. Yes
2. No
3. Identify High-Volume ReadQueries
4. Are These Queries Complex?
5. Move Queries toReplica/Warehouse
6. Simplify Queries or OptimizeOLTP Performance
7. Test Replication Strategies
8. Monitor and Adjust as Needed

