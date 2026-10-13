---
type: "practice"
difficulty: "Deep"
---

# Under READ COMMITTED you see a lost-update anomaly on a counter. Walk me through when you'd move to REPEATABLE READ vs use SELECT FOR UPDATE vs an atomic UPDATE, and the cost of each.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Payment Reconciliation](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Payment_Reconciliation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with concurrency issues like lost-update anomalies in a distributed system, deciding between different isolation levels or specific transaction mechanisms is crucial. In this scenario, you are observing a lost-update anomaly on a counter under READ COMMITTED isolation level. Let's explore the trade-offs and scenarios where moving to REPEATABLE READ, using SELECT FOR UPDATE, or implementing an atomic UPDATE might be more appropriate.

#### Triage
What is Being Probed

The interviewer is probing your understanding of database transaction isolation levels, concurrency control mechanisms, and their costs. They want to see how you would diagnose a concurrency issue in a specific context and decide on the most effective solution.

#### Assumptions
You are working with a relational database management system (RDBMS) like PostgreSQL or MySQL.
The lost-update anomaly occurs when two transactions simultaneously update a shared resource, leading to inconsistent data.
READ COMMITTED is currently in use, which means transactions can read uncommitted changes made by other transactions.
Step-by-Step Solution
1. Understanding the Problem

In READ COMMITTED, both transactions T1 and T2 could see the same initial value of the counter (e.g., 0). When T1 reads the value, it sees a snapshot of 0. Before committing its transaction, T2 also reads the value and sees 0 as well. Both transactions then increment the counter by 1, resulting in both writing 1 to the database. Therefore, the final value is 1 instead of the expected 2.

2. Isolation Level: REPEATABLE READ

REPEATABLE READ ensures that a transaction sees the same data throughout its duration, as if no other transactions are modifying it. This can prevent lost updates by making sure T2 cannot see the changes made by T1 until T1 commits and releases the lock.

When to Use: Use REPEATABLE READ when you need strong consistency across multiple reads in a transaction.
Cost: Increased overhead due to row-level locking, which can lead to performance degradation under heavy concurrency. However, it ensures that each transaction sees consistent data throughout its execution.
3. Locking Mechanism: SELECT FOR UPDATE

SELECT FOR UPDATE locks the rows selected by a query until the end of the current transaction. This prevents other transactions from modifying the locked rows until the current transaction is committed or rolled back.

When to Use: Use SELECT FOR UPDATE when you need to lock specific rows in your transaction, ensuring no other transaction can modify them.
Cost: Locks are acquired only on the rows being updated, reducing contention compared to table-level locks. However, it still incurs some overhead for acquiring and maintaining these locks.
4. Atomic UPDATE

An atomic UPDATE involves applying a single operation that updates the counter atomically without intermediate states. This can be achieved using SQL constructs like UPDATE ... SET or database-specific features like transactions with savepoints.

When to Use: Use an atomic UPDATE when you need a simple and efficient way to update a value in one step, without dealing with concurrency issues.
Cost: Minimal overhead since the operation is atomic and does not involve complex locking mechanisms. However, it might not provide the same level of transactional consistency as REPEATABLE READ.

#### Closing Summary

Each approach has its trade-offs between simplicity, performance, and data consistency:

REPEATABLE READ ensures strong transactional consistency at the cost of higher overhead.
SELECT FOR UPDATE provides a balance by locking only specific rows while maintaining some consistency guarantees.
Atomic UPDATE is simple but may not handle complex concurrency scenarios effectively.

#### Common Mistakes
and Follow-Ups
Ignoring Transaction Context: Forgetting to consider the transactional context can lead to overlooking important factors like read-write patterns.
Overlocking: Excessive use of locking mechanisms can degrade performance, especially in highly concurrent environments.
Not Considering Query Patterns: The choice should align with how data is accessed and updated.

These are common pitfalls that you should avoid when making decisions about concurrency control strategies.

