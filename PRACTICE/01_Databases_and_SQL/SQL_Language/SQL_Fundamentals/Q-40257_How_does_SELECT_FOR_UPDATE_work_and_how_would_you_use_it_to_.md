---
type: "practice"
difficulty: "Expert"
---

# How does SELECT ... FOR UPDATE work, and how would you use it to safely decrement inventory?

**Difficulty:** Expert · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ SKIP LOCKED Job Queues](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/SKIP_LOCKED_Job_Queues.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand SELECT ... FOR UPDATE in SQL, let's start by summarizing its purpose and usage. SELECT ... FOR UPDATE is a transactional locking mechanism that locks rows or ranges of rows selected from a table, preventing other transactions from modifying those rows until the current transaction completes. This is particularly useful for managing concurrent access to shared resources like inventory.

In this context, we'll explore how to use FOR UPDATE to safely decrement an inventory item. We need to ensure that two conditions are met:

The operation must be atomic and consistent.
Concurrent updates should not lead to race conditions or data inconsistencies.

Let's break down the process step by step:

#### Triaging the Question

The question asks how SELECT ... FOR UPDATE works and provides an example use case of decrementing inventory in a safe manner. This involves understanding:

Transaction Management: Ensuring that changes are committed atomically.
Row-Level Locking: Preventing other transactions from modifying the same row during the update.

#### First Principles Solution
Start a Transaction: Begin a transaction to ensure all operations are treated as a single unit of work.
Fetch and Lock Row(s): Use SELECT ... FOR UPDATE to fetch the inventory record, locking it so no other transactions can modify it until this transaction completes.
Decrement Inventory: Update the inventory count by decrementing the value.
Commit or Rollback: If the operation is successful, commit the transaction; otherwise, rollback to maintain data integrity.
Detailed Steps

Start a Transaction:

SQL
START TRANSACTION;

Fetch and Lock Row(s): Use SELECT ... FOR UPDATE to fetch the inventory record associated with a specific product ID.

SQL
SELECT * FROM Inventory WHERE ProductID = 123 FOR UPDATE;

This statement locks the row for exclusive access by this transaction.

Decrement Inventory: Update the inventory count based on the fetched record.

SQL
UPDATE Inventory SET Quantity = Quantity - 1 WHERE ProductID = 123;

Commit or Rollback:

SQL
IF (/* condition to check if decrement was successful */) THEN
    COMMIT; /* Commit transaction */
ELSE
    ROLLBACK; /* Rollback transaction if any error occurs */
END IF;

#### Closing Summary
+ Common Mistakes/Follow-Ups

#### Common Mistakes

Not starting a transaction: If multiple operations are needed, they must be grouped in a transaction.
Forgetting to lock the row(s): This can lead to race conditions and inconsistent data.
Improper error handling: Ensure proper rollback if any part of the transaction fails.

#### Follow-Ups

What happens if the product quantity is zero?
How would you handle concurrent updates from multiple transactions?

