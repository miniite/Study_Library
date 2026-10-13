---
type: "practice"
difficulty: "Intermediate"
---

# A colleague set the whole app to SERIALIZABLE "to be safe." What throughput and deadlock consequences do you warn them about?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Serializable Performance](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/Serializable_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

When a database application is configured with SERIALIZABLE isolation level, it guarantees that all transactions will behave as if they were executed in a serialized manner, which means each transaction sees a snapshot of the database as of when the transaction started. While this ensures absolute correctness and prevents most forms of concurrency issues like dirty reads or non-repeatable reads, it can significantly impact throughput and lead to deadlocks.

#### Triaging the Question
Understanding the Problem: Your colleague set the entire application to SERIALIZABLE isolation level because they wanted "to be safe." The question probes your understanding of how different isolation levels affect database performance and concurrency.

#### Assumptions
The application is currently experiencing some concurrency issues (e.g., dirty reads, non-repeatable reads).
The application’s performance has been impacted by these issues, leading to the decision to switch to SERIALIZABLE.
Step-by-Step Solution from First Principles:
1. Understanding SERIALIZABLE Isolation Level

SERIALIZABLE is the strongest isolation level in relational databases like PostgreSQL and MySQL's InnoDB. It ensures that transactions are fully isolated from each other, meaning:

No Dirty Reads: A transaction cannot read uncommitted data.
No Non-Repeatable Reads: The same query run multiple times during a transaction will see the same data.
No Phantom Reads: New rows inserted or deleted by another transaction do not appear in your transaction.
2. Throughput Impact

Switching to SERIALIZABLE can significantly reduce the number of transactions that can be executed concurrently, leading to lower throughput:

Lock Contention: Every read operation may require a lock on the data being read, which can block other transactions from reading or writing the same data.
Read Locks: Even when you do not write any data, your transaction might hold locks for a long time due to read operations, preventing others from accessing the same rows.
3. Deadlock Consequences

SERIALIZABLE isolation can increase the likelihood of deadlocks:

Cycle of Locks: Multiple transactions may each be waiting for a lock held by another transaction, creating a deadlock cycle.
Timeouts and Aborts: Deadlocks are typically resolved by rolling back one or more transactions, which can introduce additional overhead and reduce overall system availability.
4. Example Scenarios

#### Cons
ider two transactions:

Transaction A: Reads data X and holds a shared lock on it.
Transaction B: Reads the same data X and requests an exclusive lock to write, but gets blocked because of Transaction A's shared lock.
Transaction C: Writes to another row and needs a shared lock on X, which is held by Transaction A.

In this scenario, all three transactions are waiting for each other, leading to a deadlock.

5. Alternative Approaches

To mitigate these issues while still maintaining data integrity:

Reduce Locking Granularity: Use more fine-grained locks (e.g., row-level instead of table-level).
Optimize Read Operations: Minimize read operations within transactions and use caching where possible.
Use REPEATABLE READ or READ COMMITTED: These isolation levels offer a good balance between performance and correctness.

#### Closing Summary

While setting the entire application to SERIALIZABLE can prevent most concurrency issues, it comes with significant trade-offs in terms of throughput and potential deadlocks. It is advisable to carefully evaluate whether the benefits outweigh the costs by considering alternative strategies that provide similar guarantees but with better performance.

#### Common Mistakes/Follow-Ups
Mistake: Assuming SERIALIZABLE is always a safer choice without understanding its implications.

#### Follow-Up
Discussing specific database features like MVCC (Multi-Version Concurrency Control) and their impact on isolation levels and performance.

