---
type: "practice"
difficulty: "Deep"
---

# What is data skew in a distributed query and why can one heavy key make a join slow on a cluster?

**Difficulty:** Deep · **Levels:** Fresher · [↑ Composite vs Surrogate Keys](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Composite_vs_Surrogate_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In the context of distributed databases, data skew refers to a situation where data is not evenly distributed across partitions or nodes. This uneven distribution can lead to performance bottlenecks in operations like joins, especially when dealing with large datasets. In this scenario, a single key (or a few keys) may contain significantly more data than others, causing certain nodes to process much larger amounts of data than their peers.

To illustrate, imagine you have two tables: orders and customers. Normally, the distribution of customer IDs would be roughly even, allowing for balanced load across all nodes. However, if a specific customer ID (let’s call it "customer_123") has an exceptionally large number of orders, then any join operation involving this key will result in one node processing far more data than others.

Let's break down why this happens and how it affects the performance:

#### Triage

The question is asking us to understand what causes a heavy key skew during joins and its impact on distributed query performance. We need to consider the nature of distributed databases, the concept of hash partitioning or range partitioning, and the implications of uneven data distribution.

#### First Principles Solution

Hash Partitioning: In many distributed database systems, tables are often partitioned based on a hash function applied to one or more columns (keys). For instance, each row is hashed according to its key value, and then directed to a specific node. If the keys have an uneven distribution, some nodes will end up with significantly more data than others.

Range Partitioning: Similarly, range partitioning divides data based on ranges of values. Again, if certain ranges contain much more data, it can lead to skew.

Impact on Joins:

In a distributed join operation, each node is expected to perform the same amount of work.
When there's data skew, some nodes may end up processing far more data than others. For example, in an SQL query like SELECT * FROM orders JOIN customers ON orders.customer_id = customers.id, if customer_123 has many rows in the orders table, one node might be burdened with a disproportionate amount of work.

#### Example

Suppose you have 10 nodes and each should handle roughly equal amounts of data. If customer_123 has 90% of all orders, then one node will need to process most of the join operations, while others might be mostly idle.

#### Closing Summary

Data skew in a distributed query can severely impact performance by causing certain nodes to become bottlenecks. This happens when data is not evenly distributed across partitions or nodes, leading to some nodes handling much more work than others during operations like joins. Understanding this concept helps in designing databases and queries that mitigate skew, ensuring balanced workload distribution.

#### Common Mistakes/Follow-Ups
Mistake: Ignoring the possibility of skew can lead to suboptimal query performance.

#### Follow-Up
How would you detect data skew? What are some strategies to prevent or mitigate it?

To conclude, recognizing and addressing data skew is crucial for optimizing distributed database operations.

**Comparison:**

| orders | customers |
|--------|-----------|
| customer_id, order_id | id, name |
| 1, 101  | 1, Alice   |
| 2, 102  | 2, Bob     |
| 123, 900| 123, Charlie |

**Code:**

```sql
-- Example SQL query demonstrating a skewed join
SELECT *
FROM orders o
JOIN customers c ON o.customer_id = c.id;
```

**Example input / output:**

```text
-- Example input -> output for a skewed join query
Input:
SELECT *
FROM orders o
JOIN customers c ON o.customer_id = c.id;

Output (simplified):
[{
  "order_id": 101,
  "customer_id": 1,
  "name": "Alice"
},
// ... many more rows where customer_id != 123
{
  "order_id": 900,
  "customer_id": 123,
  "name": "Charlie"
}
]
```

**Data:**

```json
{
  "orders": [
    { "customer_id": 1, "order_id": 101 },
    { "customer_id": 2, "order_id": 102 },
    // ... many more orders for customer_123
    { "customer_id": 123, "order_id": 900 }
  ],
  "customers": [
    { "id": 1, "name": "Alice" },
    { "id": 2, "name": "Bob" },
    // ... many more customers, including customer_123
    { "id": 123, "name": "Charlie" }
  ]
}
```

**Flow:**

1. No
2. Yes
3. Start
4. Is the data distribution even?
5. Identify skewed key
6. Optimize query or datapartitioning to balance load
7. Load is balanced, proceed withquery execution

