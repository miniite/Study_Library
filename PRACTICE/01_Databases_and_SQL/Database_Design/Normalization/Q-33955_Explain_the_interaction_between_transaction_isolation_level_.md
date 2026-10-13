---
type: "practice"
difficulty: "Expert"
---

# Explain the interaction between transaction isolation level and maintaining a denormalized aggregate consistently.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Denormalization and Locking](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Denormalization_Trade-offs_and_Patterns/Denormalization_and_Locking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to understand how different transaction isolation levels can affect the consistency of a denormalized aggregate in a database. A denormalized aggregate is typically used to optimize read operations by reducing joins and improving performance, but it requires careful handling during write operations to maintain consistency. Transaction isolation levels determine how transactions see uncommitted changes made by other transactions.

#### Triaging the Question

The question asks us to explore the interaction between transaction isolation levels (e.g., READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, SERIALIZABLE) and maintaining a denormalized aggregate consistently. The core idea is that different isolation levels can lead to inconsistent reads of the aggregate due to uncommitted changes being visible or not.

#### First-Principles Solution

Let's break down each isolation level:

READ UNCOMMITTED: This allows a transaction to read data even if other transactions have modified it but not yet committed those modifications. In this mode, the denormalized aggregate could reflect partially updated data, leading to inconsistent results.

READ COMMITTED: A transaction can only see changes made by previously committed transactions. However, if another transaction is modifying the underlying data and hasn't committed yet, the aggregate might still be in an inconsistent state during read operations.

REPEATABLE READ: Ensures that once a value has been read, it will always return the same value until the transaction ends. This means that the denormalized aggregate will reflect consistent values, but there's a risk of phantom reads if the underlying data changes between the initial read and subsequent reads in the same transaction.

SERIALIZABLE: The strongest isolation level ensures that transactions are executed as if they were running one at a time, serially. This guarantees no visibility to uncommitted changes from other transactions, but it can lead to increased locking and reduced concurrency.

#### Example Scenario

#### Cons
ider a simple example where we have an Orders table and a denormalized aggregate in the form of a summary table OrderSummary that tracks total sales by date. Suppose two transactions are running:

Transaction A: Updates an order's status to "Shipped" but hasn't committed yet.
Transaction B: Reads the OrderSummary to get the current day's total sales.

If Transaction B runs with a lower isolation level (e.g., READ UNCOMMITTED), it might see the uncommitted update from Transaction A, leading to an inconsistent summary.

#### Closing Summary

To maintain consistency in a denormalized aggregate, higher transaction isolation levels are generally required. However, this comes at the cost of reduced concurrency and potential performance bottlenecks due to increased locking. The choice of isolation level depends on the trade-offs between consistency, performance, and concurrency needed for your specific use case.

#### Common Mistakes
Failing to Consider Isolation Levels: Many developers might overlook how transaction isolation levels affect read operations on denormalized aggregates.
Overlooking Phantom Reads: Even with REPEATABLE READ or SERIALIZABLE, the risk of phantom reads still exists if the underlying data changes between the initial and subsequent reads.

#### Follow-Up
Questions
How would you design a schema that balances consistency and performance?
What are some strategies to handle write-heavy scenarios where maintaining high isolation levels is impractical?

**Comparison:**

| Date       | Total Sales |
|------------|-------------|
| 2023-10-01 | $1000.50    |

**Code:**

```sql
-- Example of an OrderSummary table with denormalized data
CREATE TABLE OrderSummary (
    date DATE,
    total_sales DECIMAL(10, 2)
);

-- Insertion in a lower isolation level scenario (e.g., READ UNCOMMITTED)
BEGIN TRANSACTION;
UPDATE Orders SET status = 'Shipped' WHERE order_id = 12345; -- Uncommitted change
SELECT SUM(amount) INTO @total FROM OrderItems WHERE date >= CURDATE();
UPDATE OrderSummary SET total_sales = @total WHERE date = CURDATE(); -- Potential inconsistency if read uncommitted
COMMIT;
```

**Example input / output:**

```text
Transaction A:
BEGIN TRANSACTION;
UPDATE Orders SET status = 'Shipped' WHERE order_id = 67890; -- Not committed yet

Transaction B (READ UNCOMMITTED):
SELECT SUM(amount) INTO @total FROM OrderItems WHERE date >= CURDATE();
-- @total might include the uncommitted change from Transaction A, leading to inconsistency
```

**Data:**

```json
{
  "Orders": [
    { "order_id": 12345, "status": "Shipped" },
    { "order_id": 67890, "status": "Pending" }
  ],
  "OrderSummary": {
    "date": "2023-10-01",
    "total_sales": 1000.50
  }
}
```

**Flow:**

1. READ UNCOMMITTED
2. READ COMMITTED
3. REPEATABLE READ
4. SERIALIZABLE
5. Transaction A
6. Update Order Status
7. Isolation Level
8. Reads Uncommitted Changes
9. Sees Committed Changes Only
10. Consistent Reads, Risk ofPhantom Reads
11. No Uncommitted Changes,High Locking

