---
type: "practice"
difficulty: "Deep"
---

# How would you keep a wallet balance correct if two payment requests arrive at the same moment?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Payment Reconciliation](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Payment_Reconciliation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure the wallet balance remains correct when two payment requests arrive simultaneously, we need to address the issue of concurrency and race conditions in our system design. This problem can be approached from a high-level perspective by considering both transactional integrity and performance efficiency.

The core question here is how to handle concurrent updates to a shared resource (the wallet balance) without causing data corruption or inconsistencies. Let's break down the solution step-by-step:

#### Triaging the Question
Problem Statement: Two payment requests arrive at nearly the same time, leading to potential race conditions.
What’s Being Probed: Understanding of concurrency control, thread safety, and database transactions.

#### Assumptions
The system uses a shared resource (wallet balance).
Each request contains transaction details (amount, user ID, etc.).
We are using a relational database to store wallet balances.

#### First Principles Solution
Read-Modify-Write Pattern: This pattern ensures that the state is read from memory, modified in a local cache or temporary storage, and then written back to the database atomically.
Locking Mechanisms: Utilize locking mechanisms (row-level locks) during transactions to prevent multiple concurrent updates to the same resource.
Detailed Steps

Database Transaction Isolation Levels:

Use Serializable isolation level for ensuring that the transaction is completely isolated from other transactions, preventing any dirty reads, writes, or commits.

Locking Mechanisms:

Implement row-level locks on the wallet balance record to prevent simultaneous modifications.

Optimistic vs Pessimistic Locking:

Pessimistic Locking: Acquire a lock before starting a transaction and release it after committing or rolling back.
SQL
-- Example SQL for pessimistic locking
SELECT * FROM wallets WHERE user_id = 1 FOR UPDATE;
Optimistic Locking: Assume no conflict until the end of the transaction. If a conflict is detected, roll back and retry.

Concurrency Control Algorithms:

Two-Phase Locking (2PL): Ensure that once a lock is acquired, it remains until the transaction ends.
Transaction Starts
Acquire Row-Level Lock
Modify Wallet Balance
Check for Conflicts
(No Conflict)
Commit Transaction
(Conflict Detected)
Rollback and Retry

Database Concurrency Control Features:

Leverage database features like SELECT ... FOR UPDATE or SAVEPOINT for managing transactional consistency.

Fail-Safe Mechanisms:

Implement retry logic in the application to handle transient failures due to locks.

#### Closing Summary

To keep a wallet balance correct during concurrent payment requests, we need to implement robust concurrency control mechanisms. This includes using appropriate isolation levels, locking strategies, and fail-safe mechanisms like retries. The key is to ensure that the system can handle race conditions without compromising data integrity or user experience.

#### Common Mistakes/Follow-Ups
Not Using Proper Locking: Failing to use locks can lead to data corruption.
Choosing Inappropriate Isolation Level: A too low isolation level can cause dirty reads, while a too high one (like Serializable) can introduce unnecessary overhead.
Ignoring Retry Logic: Without proper retry logic, transient failures might result in inconsistent states.

