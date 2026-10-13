---
type: "practice"
difficulty: "Deep"
---

# How do foreign key checks acquire locks and cause unexpected contention on the parent table during inserts?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Lock Holding and Optimistic Concurrency](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Lock_Holding_and_Optimistic_Concurrency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When inserting a record into a child table that has a foreign key constraint, the database system must ensure referential integrity by checking if the referenced value in the parent table exists. This process can lead to unexpected contention on the parent table because of how locks are acquired and managed during these foreign key checks.

#### Triaging the Question

The question is probing your understanding of transactional consistency and locking mechanisms in databases, particularly focusing on foreign key constraints and their impact on performance. It asks you to explain how locks are acquired during such operations and why this can cause contention issues.

#### First-Principles Solution
Locking Mechanism Overview

Locks are essential for maintaining data integrity and concurrency control within a database transaction. When a transaction inserts or updates a record in the child table, it needs to check if the referenced value exists in the parent table. This involves acquiring locks on the relevant rows of the parent table.

Acquiring Locks During Foreign Key Checks
Lock Request: When an insert operation occurs in the child table with foreign key constraints, the database engine first requests a lock on the corresponding row or rows in the parent table.
Row-Level Locking: Typically, this is done using row-level locks to prevent other transactions from modifying or deleting the referenced data during the check.
Lock Contention: If another transaction holds an incompatible lock (e.g., a write lock) on these rows of the parent table, it can cause contention, leading to delays in acquiring the necessary locks.

#### Example Scenario

#### Cons
ider two tables: Orders and Customers. An insert operation is attempted in the Orders table for a new order with a customer ID that does not exist yet. The database needs to check if this customer ID exists in the Customers table:

SQL
INSERT INTO Orders (order_id, customer_id) VALUES (1001, 9999);
Lock Request: The insert operation requests locks on all rows in the Customers table with a customer_id of 9999.
Contention Issue: If another transaction is modifying or inserting records into the Customers table for ID 9999, it will hold a lock. This prevents the new order from being inserted until that transaction releases its locks.
Performance Impact

This locking mechanism can lead to significant performance degradation in scenarios where many concurrent transactions are trying to insert data with foreign key constraints. The parent table may become a bottleneck due to the high number of lock requests and potential contention.

#### Closing Summary
+ Common Mistakes/Follow-Ups

#### Common Mistakes
Misunderstanding the role of locks in ensuring referential integrity can lead to inefficient database designs.

#### Follow-Up
Questions: How does indexing affect this process? Are there any configurations or optimizations that could reduce lock contention?
Alternative Approaches: Discussing techniques such as deferred foreign key constraints, optimistic locking, or partitioning tables can provide insights into mitigating these issues.

#### Triaging the Question

In summary, foreign key checks during inserts require locks on the parent table, which can cause unexpected contention. Understanding this process and its impact is crucial for designing efficient database systems that handle concurrent transactions effectively.

**Comparison:**

```
table "Orders" as o
table "Customers" as c

o-->c: customer_id 9999
```

**Code:**

```sql
-- Example SQL operation with a foreign key constraint
INSERT INTO Orders (order_id, customer_id) VALUES (1001, 9999);
```

**Example input / output:**

```text
Input:
INSERT INTO Orders (order_id, customer_id) VALUES (1001, 9999);

Output:
Lock request on Customers table for customer_id 9999.
Contention check with another transaction modifying the same row.
```

**Data:**

```json
{
  "Orders": [
    {
      "order_id": 1001,
      "customer_id": 9999
    }
  ],
  "Customers": [
    {
      "customer_id": 1,
      "name": "Alice"
    },
    {
      "customer_id": 2,
      "name": "Bob"
    }
  ]
}
```

**Flow:**

1. +-----------------------+
2. | Insert Order (1001)   |
3. +--------+------------+
4. |
5. v
6. +--------v------------+
7. | Lock Request on     |
8. | Customers (9999)   |
9. +--------+------------+
10. |
11. v
12. +--------v------------+
13. | Contention Check   |
14. | with Another T.    |
15. +-------------------+

