---
type: "practice"
difficulty: "Deep"
---

# How do you decide isolation for a system where the same data is written by both a real-time API and a nightly bulk ETL that MERGEs millions of rows? How do you keep them from anomaly interference?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Isolation for Shared Subsystems](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/Isolation_for_Shared_Subsystems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding the appropriate isolation level in a system where both a real-time API and a nightly bulk ETL are updating the same data, the key is to balance between performance and data integrity. The isolation levels (RU, RC, RR, and Serializable) each come with their own trade-offs that need careful consideration. In this scenario, we must ensure that neither process interferes with the other while maintaining acceptable read and write latency.

#### Triage

The core of the question is about determining the right isolation level to minimize conflicts between real-time updates and bulk ETL operations. Specifically:

Real-Time API: Frequent, potentially high-concurrency writes.
Nightly ETL: Large-scale writes that can take a long time but are less frequent.
Key Concepts
RU (Read Uncommitted): No transaction isolation; can lead to dirty reads and other anomalies.
RC (Read Committed): Prevents dirty reads, but still allows non-repeatable reads and phantom reads.
RR (Repeatable Read): Ensures repeatable reads within a transaction but may not be sufficient for bulk ETL operations.
Serializable: Highest level of isolation, ensuring no anomalies but can degrade performance significantly.

#### First-Principles Solution

Understand the Data Write Patterns:

Real-time API: Likely has frequent small writes or updates.
Nightly ETL: Large bulk merges without strict order requirements.

Impact on Read Operations:

For real-time reads, we need to ensure that they are not impacted by dirty data from ongoing transactions.
The nightly ETL should be able to run efficiently without being slowed down by frequent read operations.

Isolation Level Selection:

RC (Read Committed): Ensures that readers see only committed changes. This minimizes the risk of anomalies but might still cause non-repeatable reads and phantom reads.
RR (Repeatable Read): Better ensures repeatable reads, reducing the risk of read anomalies.

Transaction Management:

Use explicit transactions for both real-time API updates and ETL processes to manage consistency levels.
Ensure that ETL operations can run in long-running transactions if necessary, but use appropriate isolation settings to minimize impact on real-time reads.

Concurrency Control:

Implement locking strategies (optimistic vs. pessimistic) based on the nature of the operations and their frequency.
Use row-level locks for real-time API updates to prevent other transactions from modifying the same rows concurrently.

#### Closing Summary

For this system, a balanced approach with Read Committed (RC) isolation level is likely the best choice. It ensures that reads do not see uncommitted data while still allowing efficient bulk ETL operations without excessive overhead. However, fine-grained locking mechanisms and transaction management should be carefully designed to minimize contention between the real-time API and ETL processes.

#### Common Mistakes/Follow-Ups
Overusing Serializable: This can lead to significant performance degradation due to increased locking and reduced concurrency.
Ignoring Concurrency Control: Improper handling of locks and transactions can cause data anomalies or bottlenecks.
Neglecting Performance Monitoring: Continuous monitoring is essential to identify and mitigate performance issues caused by isolation settings.

**Comparison:**

```
OrderID	CustomerID	Quantity
12345	67890	10
OrderID	NewQuantity
12345	10
```

**Code:**

```sql
-- Example SQL for RC Isolation with Transactions

BEGIN TRANSACTION;

-- Real-time API Write
UPDATE Orders SET Quantity = 10 WHERE OrderID = 12345;

-- Nightly ETL Merge (with appropriate isolation)
MERGE INTO Orders o
USING (
    SELECT * FROM TemporaryOrders
) t
ON o.OrderID = t.OrderID
WHEN MATCHED THEN
    UPDATE SET o.Quantity = t.NewQuantity
WHEN NOT MATCHED THEN
    INSERT (OrderID, Quantity) VALUES (t.OrderID, t.NewQuantity);

COMMIT;
```

**Example input / output:**

```text
**Dry-run Example**

- **Input**:
  - Real-time API update: `UPDATE Orders SET Quantity = 10 WHERE OrderID = 12345;`
  - Nightly ETL merge: `MERGE INTO Orders o USING TemporaryOrders t ON o.OrderID = t.OrderID...`

- **Output**:
  - After real-time update, `Quantity` for Order ID 12345 is updated to 10.
  - ETL operation merges and updates the row with new quantity.
```

**Data:**

```json
{
  "Orders": [
    {
      "OrderID": 12345,
      "CustomerID": 67890,
      "Quantity": 5
    }
  ],
  "TemporaryOrders": [
    {
      "OrderID": 12345,
      "NewQuantity": 10
    }
  ]
}
```

**Flow:**

1. Real-time API Update
2. Isolation Level?
3. Read Committed
4. Nightly ETL Merge
5. Locking Strategy?
6. Pessimistic Locks
7. Transaction Commit
8. Data Consistency

