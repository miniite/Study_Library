---
type: "practice"
difficulty: "Intermediate"
---

# An application relies on autocommit and each INSERT is its own transaction, hurting throughput. How do you batch commits safely without risking partial loss?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Transaction Control and Prepared Transactions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Transaction_Control_and_Prepared_Transactions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When an application uses autocommit and each INSERT operation is a separate transaction, it can indeed hurt database throughput due to the overhead of starting and committing each transaction individually. To improve performance while ensuring data integrity, you need to batch these operations into transactions that commit together. This approach balances between frequent commits and potential risks like partial losses.

To address this issue safely, we'll:

Understand the current setup.
Introduce transaction management techniques.
Ensure data consistency through proper handling of failures.
Optimize for performance without compromising reliability.

#### Triaging the Question

The core question is about optimizing database throughput in a scenario where each INSERT operation is currently wrapped in its own transaction, leading to high overhead. The interviewer is probing your understanding of:

Autocommit and how it impacts performance.
Transaction management techniques such as batch commits.
Data consistency and handling failures during partial transactions.

#### First-Principles Solution
Review Current Setup: Confirm that the application uses autocommit with each INSERT being a separate transaction. This means every INSERT is committed immediately, which can be costly in terms of performance due to frequent context switches between the database and your application.
Batch Inserts into Transactions:
Group multiple INSERT statements into a single transaction using explicit start and commit commands (e.g., BEGIN, COMMIT).
This reduces the overhead by minimizing the number of commits required.
Ensure Atomicity and Isolation:
Use transaction isolation levels to manage concurrency issues and ensure data consistency.
Ensure that transactions are idempotent or handle partial failures gracefully.
Error Handling and Recovery:
Implement robust error handling within transactions to catch and recover from failures.
Log errors and retries appropriately to avoid data loss.
Detailed Steps
Review Application Code: Check the current implementation of INSERT statements to confirm they are using autocommit.
Add Transaction Boundaries:
SQL
BEGIN;
-- Multiple INSERT statements here
COMMIT;
Adjust Isolation Levels (optional but recommended):
Set appropriate transaction isolation levels in your database configuration or within the transactions themselves to handle concurrent access properly.
Error Handling:
Implement try-catch blocks around transactions to catch and log errors.
Use stored procedures or functions to encapsulate complex operations with error handling.
Test Thoroughly: Ensure that batching does not introduce new issues, especially under high load.

#### Closing Summary
+ Common Mistakes

By switching from autocommit to batched transactions, you can significantly improve throughput. However, it's crucial to handle failures properly to avoid partial data loss:

Missteps might include forgetting to commit or rollback transactions, leading to partial states.
Not handling errors within transactions can result in lost operations if the transaction fails midway.

#### Common Mistakes/Follow-Ups
Not Testing Thoroughly: Always test batched transactions under various load conditions and failure scenarios to ensure data integrity.
Ignoring Rollback Logic: Ensure that any failed transaction has a proper rollback mechanism to maintain consistency.
Overlooked Concurrency Issues: Make sure to address potential concurrency issues by adjusting isolation levels or using locking mechanisms.

**Code:**

```sql
BEGIN;
INSERT INTO orders (customer_id, product_id, quantity) VALUES (101, 201, 5);
INSERT INTO orders (customer_id, product_id, quantity) VALUES (102, 202, 3);
COMMIT;
```

**Example input / output:**

```text
Input:
BEGIN;
INSERT INTO orders (customer_id, product_id, quantity) VALUES (101, 201, 5);
INSERT INTO orders (customer_id, product_id, quantity) VALUES (102, 202, 3);
COMMIT;

Output:
Transaction successfully completed.
```

**Flow:**

1. Start Transaction
2. Perform Multiple Inserts
3. Commit Transactions orRollback on Error
4. End Transaction
5. Handle Errors, Log and Retry ifNecessary

