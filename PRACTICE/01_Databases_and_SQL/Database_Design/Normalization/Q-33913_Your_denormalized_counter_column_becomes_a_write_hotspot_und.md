---
type: "practice"
difficulty: "Expert"
---

# Your denormalized counter column becomes a write hotspot under high concurrency. How would you redesign to avoid lock contention?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Denormalization and Locking](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Denormalization_Trade-offs_and_Patterns/Denormalization_and_Locking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a write hotspot in a denormalized counter column due to high concurrency, we need to rethink our database design to balance performance and consistency. The core idea is to introduce a more robust state management mechanism that can handle concurrent writes without causing lock contention.

#### Triaging the Question

The question asks us to redesign a system where a single counter column is being frequently updated, leading to significant write contention. This implies that we are dealing with a classic case of high concurrency and need to optimize our schema or introduce additional mechanisms to manage these updates effectively.

Key Assumptions:
The current design uses a denormalized approach, storing the count directly in a single column.
High concurrency is causing significant lock contention on this counter column.
The application needs to maintain some level of consistency and accuracy for the counter value.

#### Solution
Approach

The first step is to understand why a write hotspot occurs. In high-concurrency scenarios, multiple transactions trying to update the same row simultaneously can lead to frequent locking or even deadlocks. To avoid this, we need to explore alternative strategies like:

Atomic Operations: Using atomic operations that allow us to increment the counter in a single operation.
Multi-Row Transactions: Ensuring that all necessary rows are updated atomically within a transaction.
Optimistic Concurrency Control (OCC): Implementing versioning or other forms of optimistic concurrency control to avoid conflicts.
Step-by-Step Solution
Atomic Increment Operation:
Use SQL functions like UPDATE ... SET with an expression that increments the counter atomically.

#### Example
in PostgreSQL:
SQL
UPDATE my_table SET counter = counter + 1 WHERE id = ? RETURNING counter;
Multi-Row Transactions:
Ensure that related data is updated within a single transaction to avoid partial updates and inconsistencies.
Optimistic Concurrency Control (OCC):
Add a version column to track changes and use it in the update query to ensure only consistent updates are applied.
SQL
UPDATE my_table SET counter = counter + 1, version = version + 1 WHERE id = ? AND version = ?
RETURNING counter;
Read-After-Writes:
Implement a mechanism where read operations can refresh the cache or pull in the latest values before performing another write.

#### Closing Summary

By applying these strategies, we can significantly reduce lock contention and improve the performance of our system under high concurrency. The choice of which strategy to implement depends on the specific requirements and constraints of the application.

#### Common Mistakes
and Follow-Ups:
Misunderstanding Atomicity: Failing to recognize that certain operations need to be atomic.
Ignoring Concurrency Control Mechanisms: Not considering OCC or other concurrency control strategies can lead to partial updates or inconsistencies.
Overlooking Transaction Management: Ensuring all related data is updated within a single transaction is crucial but often overlooked.

**Code:**

```sql
-- Atomic Increment Operation Example in PostgreSQL
UPDATE my_table SET counter = counter + 1 WHERE id = ? RETURNING counter;

-- Optimistic Concurrency Control Example
UPDATE my_table SET counter = counter + 1, version = version + 1 WHERE id = ? AND version = ?
RETURNING counter;
```

**Example input / output:**

```text
-- Initial State
SELECT * FROM my_table WHERE id = 1;

-- Increment Counter with Atomic Operation
UPDATE my_table SET counter = counter + 1 WHERE id = 1 RETURNING counter;

-- Verify the New Count
SELECT * FROM my_table WHERE id = 1;
```

**Flow:**

1. No
2. Yes
3. Transaction Begins
4. Is Version Equal?
5. Fetch Latest Counter
6. Update Counter and IncrementVersion
7. Commit Transaction
8. Write Conflicts Detected
9. Retry Operation

