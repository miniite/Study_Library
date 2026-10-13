---
type: "practice"
difficulty: "Deep"
---

# A nightly reconciliation job and live traffic contend on the same rows. How would you schedule or isolate the job to remove the contention without stale results?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Row Locks Two-phase Locking and Lock Footprint](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Row_Locks_Two-phase_Locking_and_Lock_Footprint.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the problem of a nightly reconciliation job contending with live traffic on the same database rows, we need to ensure that the reconciliation job can run without causing stale data or deadlocks. The key challenge here is isolating the reconciliation process from concurrent transactions while still ensuring that it runs accurately and doesn't miss any changes made during its execution.

#### Triage

The question asks how to schedule or isolate a nightly reconciliation job so that it does not conflict with ongoing live traffic on the same database rows. This involves understanding concurrency control, locking strategies, and scheduling mechanisms. The interviewer is likely probing your knowledge of transaction isolation levels, locking mechanisms, and how to manage concurrent access in a database environment.

#### Assumptions
Database System: We are working with a relational database management system (RDBMS) that supports various locking mechanisms.
Concurrency Levels: There are ongoing transactions from live traffic that may be accessing the same rows as the reconciliation job.
Isolation Requirements: The reconciliation process needs to ensure data consistency and accuracy, which might involve strong isolation levels.

#### First-Principles Solution

To resolve this issue, we can employ a combination of techniques such as scheduled maintenance windows, locking granularity, transaction isolation levels, and possibly background processing or queuing mechanisms. Here’s a step-by-step breakdown:

Scheduled Maintenance Window: Schedule the reconciliation job during off-peak hours when live traffic is minimal.
Transaction Isolation Levels: Use appropriate isolation levels to manage concurrent transactions safely.
Locking Mechanisms: Implement fine-grained locking or use optimistic concurrency control where possible.
Queue-Based Processing: Enqueue all changes and process them in a batch outside of the live transaction window.
Detailed Solution
1. Scheduled Maintenance Window

The most straightforward approach is to schedule the reconciliation job during periods when there is minimal live traffic. For example, if the system experiences high usage during business hours, scheduling the job between midnight and early morning might be effective. This ensures that the database is less busy, reducing contention.

2. Transaction Isolation Levels

Set the transaction isolation level to SERIALIZABLE or a similar high-isolation mode to ensure that no concurrent transactions can modify the same rows during the reconciliation process. However, this may impact performance due to increased locking overhead.

SQL
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
3. Locking Mechanisms

Implement fine-grained row-level locks or use optimistic concurrency control (OCC) techniques where possible. This can help manage concurrent access more efficiently than full table locks, which could block all other transactions.

#### Example
of explicit locking:

SQL
BEGIN TRANSACTION;

SELECT * FROM orders WITH (ROWLOCK, UPDLOCK);

-- Perform reconciliation operations here

COMMIT;
4. Queue-Based Processing

Another approach is to use a queue-based system where changes are recorded and processed in batches outside the live transaction window. This can be implemented using a message broker like Apache Kafka or RabbitMQ.

Capture Changes: Live transactions record their changes into an audit table.
Process Changes: Reconciliation job reads from this audit table, processes the data, and updates the database in bulk operations.
SQL
-- Example of capturing changes
INSERT INTO order_changes (order_id, status) VALUES (...);

-- Example of processing changes
BEGIN TRANSACTION;

SELECT * FROM order_changes;

UPDATE orders SET status = 'processed' WHERE id IN (SELECT order_id FROM order_changes);

COMMIT;

#### Closing Summary

By employing a combination of scheduled maintenance windows, appropriate transaction isolation levels, fine-grained locking mechanisms, and queue-based processing, we can effectively manage the concurrency between live traffic and reconciliation jobs without causing stale results.

#### Common Mistakes/Follow-Ups
Over-reliance on Locking: Using too many or overly broad locks can lead to performance degradation.
Insufficient Isolation Levels: Not setting strong enough isolation levels can result in inconsistent data.
Ignoring Off-peak Scheduling: Failing to schedule the job during off-peak times can still cause contention and delays.

#### Follow-ups
might include asking about specific locking strategies, trade-offs between different isolation levels, or how to monitor and optimize performance during maintenance windows.

**Comparison:**

```
Order ID	Status
101	shipped
102	cancelled
```

**Code:**

```sql
-- Example of setting transaction isolation level
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Example of explicit row-level locking
BEGIN TRANSACTION;
SELECT * FROM orders WITH (ROWLOCK, UPDLOCK);
UPDATE orders SET status = 'processed' WHERE id IN (SELECT order_id FROM order_changes);
COMMIT;
```

**Example input / output:**

```text
Input:
- Live transaction records order changes in `order_changes` table.
- Reconciliation job processes these changes and updates the `orders` table.

Output:
- `orders` table is updated with processed statuses from live transactions.
```

**Data:**

```json
{
  "order_changes": [
    {"order_id": 101, "status": "shipped"},
    {"order_id": 102, "status": "cancelled"}
  ]
}
```

**Flow:**

1. Live Transaction
2. Capturing changes inorder_changes
3. Reconciliation Job
4. Processing changes fromorder_changes
5. Order Changes Table
6. Updating orders table withprocessed statuses

