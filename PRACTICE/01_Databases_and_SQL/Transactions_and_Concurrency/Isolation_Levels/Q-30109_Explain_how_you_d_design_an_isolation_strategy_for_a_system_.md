---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd design an isolation strategy for a system that must satisfy both a hard regulatory audit (immutable consistent history) and low-latency serving.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Isolation for Shared Subsystems](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/Isolation_for_Shared_Subsystems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an isolation strategy for a system that needs to satisfy both a hard regulatory audit requirement of immutable, consistent historical data and achieve low-latency serving, we need to understand how different isolation levels (Read Uncommitted (RU), Read Committed (RC), Repeatable Read (RR), and Serializable) affect the system. The key is balancing consistency with performance.

#### Triage

The question is essentially asking us to design a database strategy that ensures data integrity and regulatory compliance while maintaining low-latency operations. This involves understanding the trade-offs between isolation levels, the specific requirements of each, and how they can be balanced.

#### Assumptions
The system handles sensitive data.
There are strict regulatory requirements for data immutability and consistency.
Low latency is critical for serving real-time queries or transactions.

#### First-Principles Solution

Isolation levels determine the degree of isolation between transactions in a database. Here’s how each level affects our scenario:

Read Uncommitted (RU): This allows reading uncommitted data, which can lead to dirty reads and make it impossible to satisfy regulatory requirements.
Read Committed (RC): This ensures that only committed data is read. While this meets the regulatory requirement for immutability, it does not guarantee repeatable reads or serializability, leading to potential performance issues in concurrent operations.
Repeatable Read (RR): This enforces repeatable reads and prevents non-repeatable reads. It also avoids phantom reads but still allows for certain types of dirty reads. RR can be sufficient if we implement additional constraints like row versioning or optimistic concurrency controls.
Serializable: This ensures that all transactions appear to execute serially, preventing any form of concurrent operations from conflicting with each other. However, it is the most restrictive and can significantly impact performance.

Given these considerations, a balanced approach would be to use RR while implementing additional measures for low latency.

#### Implementation
Strategy
Use Repeatable Read (RR): Ensure that transactions are isolated in such a way that they see consistent data throughout their execution.
Row Versioning: Implement row versioning to allow multiple versions of the same row at different points in time, which can be read by transactions with RR isolation level without causing conflicts.
Optimistic Concurrency Control (OCC): Use OCC to handle conflicts during commit times instead of within each transaction. This reduces lock contention and improves performance.
Partitioning: Partition the data based on frequently accessed parts or historical data to reduce the scope of transactions that require RR isolation.
Read-Only Transactions: For low-latency serving, mark read-only transactions as such to avoid unnecessary locking.

#### Example

#### Cons
ider a scenario where we have a database with multiple tables: Orders and OrderDetails. We want to ensure that once an order is placed, it cannot be modified (immutable history) but we need to serve real-time updates efficiently.

SQL
-- Enable row versioning for the Orders table
ALTER TABLE Orders SET (SYSTEM_VERSIONING = ON (HISTORY_TABLE = dbo.OrdersHistory));

-- Insert a new order with RR isolation
BEGIN TRANSACTION;
SET TRANSACTION ISOLATION LEVEL READ REPEATABLE;

INSERT INTO Orders (CustomerID, OrderDate) VALUES (@customer_id, GETDATE());

COMMIT;

#### Performance Considerations
Row Versioning: This allows concurrent transactions to read older versions of the data while newer changes are made.
Lock Contention: Use lock hints and transaction isolation levels judiciously to minimize blocking and deadlocks.

#### Closing Summary
+ Common Mistakes/Follow-Ups

The key is finding a balance between ensuring regulatory compliance through RR isolation and achieving low-latency performance with techniques like row versioning, OCC, and partitioning. Common mistakes include over-optimizing for one aspect at the expense of another, such as implementing RR too liberally or failing to address lock contention effectively.

In follow-up discussions, we might explore more advanced topics such as distributed transaction management (e.g., two-phase commit) and how they interact with our isolation strategy in a multi-database environment.

**Comparison:**

| CustomerID | OrderDate        |
|------------|------------------|
| 12345      | 2023-10-10 14:00 |
| ...        | ...              |

**Code:**

```sql
-- Example of row versioning setup
ALTER TABLE Orders SET (SYSTEM_VERSIONING = ON (HISTORY_TABLE = dbo.OrdersHistory));

-- Example of transaction with RR isolation level
BEGIN TRANSACTION;
SET TRANSACTION ISOLATION LEVEL READ REPEATABLE;

INSERT INTO Orders (CustomerID, OrderDate) VALUES (@customer_id, GETDATE());

COMMIT;
```

**Example input / output:**

```text
Input:
BEGIN TRANSACTION;
SET TRANSACTION ISOLATION LEVEL READ REPEATABLE;

INSERT INTO Orders (CustomerID, OrderDate) VALUES (12345, '2023-10-10');

COMMIT;

Output:
Transaction completed successfully.
```

**Data:**

```json
{
  "Orders": {
    "table_schema": "dbo",
    "columns": [
      {"name": "OrderID", "type": "int"},
      {"name": "CustomerID", "type": "int"},
      {"name": "OrderDate", "type": "datetime"}
    ],
    "history_table": "OrdersHistory"
  }
}
```

**Flow:**

1. Read Uncommitted
2. Read Committed
3. Repeatable Read
4. Serializable
5. Start Transaction
6. Isolation Level?
7. Allow Dirty Reads
8. Ensure Consistent Read
9. Prevent Non-Repeatable Reads
10. Ensure Serial Execution
11. Use Row Versioning
12. Insert New Order
13. Commit Transaction

