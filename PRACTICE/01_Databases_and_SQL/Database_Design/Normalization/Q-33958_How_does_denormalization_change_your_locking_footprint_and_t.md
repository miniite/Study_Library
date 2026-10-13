---
type: "practice"
difficulty: "Deep"
---

# How does denormalization change your locking footprint and the risk of deadlocks in a high-write system?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Denormalization and Locking](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Denormalization_Trade-offs_and_Patterns/Denormalization_and_Locking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Denormalization, often used to improve query performance by reducing join operations, has significant implications on the locking footprint and deadlock risks in a high-write system. In this context, we need to carefully consider how denormalized data structures can affect transactional behavior and concurrency.

#### Triaging the Question

The question asks us to analyze how denormalization influences the locking strategy and deadlocks in a database with frequent write operations. This involves understanding the underlying mechanisms of locks, how they interact with read and write transactions, and how denormalizing data might alter these interactions.

Key Assumptions
High Write Volume: The system experiences a significant number of write operations.
Transaction Management: Transactions in this database need to handle both reads and writes efficiently.
Deadlock Risk: Deadlocks occur when two or more transactions are waiting for each other to release locks, leading to an indefinite wait state.
Step-by-Step Solution
1. Understanding Locking Mechanisms

In a normalized database, data is spread across multiple tables, which means that reads and writes often require lock acquisition on multiple rows and possibly different tables. Denormalization combines related data into a single table or reduces the number of join operations required for queries.

2. Impact on Write Operations

When performing write operations in a denormalized schema:

Increased Locking Footprint: Since more data is stored in fewer tables, write operations may lock larger portions of the database.
Transaction Isolation Levels: Higher isolation levels (e.g., SERIALIZABLE) will further increase the locking footprint as they require more granular locks.
3. Concurrency and Deadlocks
Reduced Joins, Increased Locks: Denormalization often reduces the number of joins required to retrieve data but increases the likelihood that a single transaction needs to hold multiple locks.
Deadlock Scenarios:
Scenario A: Transaction T1 starts a write operation on Table A and then needs to read from Table B. At the same time, Transaction T2 starts a write on Table B and needs to read from Table A. Both transactions will wait indefinitely if they hold exclusive locks.
Scenario B: In a denormalized schema where data is spread across fewer tables, one transaction might block another if both try to lock different parts of the denormalized structure at the same time.
4. Example Scenario

#### Cons
ider an e-commerce system with products and orders. A normalized schema would have separate products and orders tables. In a denormalized schema, you might add order details directly into the product table:

SQL
CREATE TABLE products (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    price DECIMAL(10, 2),
    last_ordered_at TIMESTAMP NULL, -- added for denormalization
    ordered_count INT NULL        -- added for denormalization
);

When a user places an order:

A transaction writes the orders table and updates last_ordered_at in products.
Another transaction might also update ordered_count in products.

Both transactions might lock different parts of the products table, leading to potential deadlocks.

5. Mitigation Strategies
Locking Granularity: Use finer-grained locking where possible.
Serializable Isolation Levels: Be cautious as they can increase deadlock risk but ensure data consistency.
Transaction Management: Implement techniques like lock acquisition order rules and timeout settings to reduce deadlocks.

#### Closing Summary
and Common Mistakes

Denormalization in a high-write system increases the locking footprint due to fewer join operations, which can lead to more extensive locks being held by transactions. This, combined with reduced isolation levels, can significantly increase the risk of deadlocks. Proper transaction management techniques are crucial to mitigate these risks.

#### Common Mistakes/Follow-Ups
Ignoring Locking Impacts: Many developers focus solely on query performance without considering the increased locking overhead.
Overdenormalization: Going too far with denormalization can lead to data inconsistency issues and harder-to-maintain schemas.
Improper Deadlock Management: Not having robust strategies for managing deadlock scenarios can result in unhandled indefinite waits.

**Code:**

```

-- Example of a normalized schema CREATE TABLE products ( id INT PRIMARY KEY, name VARCHAR(255), price DECIMAL(10, 2) );

CREATE TABLE orders ( id INT PRIMARY KEY, product_id INT, order_time TIMESTAMP, FOREIGN KEY (product_id) REFERENCES products(id) );

CODE
Copy

```sql
-- Denormalized schema example
CREATE TABLE denormalized_products (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    price DECIMAL(10, 2),
    last_ordered_at TIMESTAMP NULL,
    ordered_count INT NULL
);
```

