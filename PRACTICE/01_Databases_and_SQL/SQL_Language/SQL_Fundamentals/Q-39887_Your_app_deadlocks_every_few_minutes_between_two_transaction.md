---
type: "practice"
difficulty: "Deep"
---

# Your app deadlocks every few minutes between two transactions. Walk through reading the deadlock graph, identifying the lock-ordering conflict, and the code or index change you'd ship.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ SKIP LOCKED Job Queues](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/SKIP_LOCKED_Job_Queues.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a database application experiences deadlocks, it's crucial to understand the underlying issues by analyzing the deadlock graph. A deadlock occurs when two or more transactions are waiting indefinitely for resources held by each other. In this scenario, we'll walk through reading a deadlock graph, identifying the lock-ordering conflict, and proposing changes to resolve the issue.

#### Triage
Understanding the Question

The question asks us to analyze a deadlock situation in an application where deadlocks occur every few minutes between two transactions. We need to read the deadlock graph, identify the specific lock-ordering conflict causing the deadlock, and suggest code or index changes that could prevent future occurrences. This involves understanding transaction locking mechanisms, analyzing the deadlock graph, and proposing optimal database schema modifications.

#### First-Principles Solution
Read the Deadlock Graph: The deadlock graph will provide visual information about the transactions involved in the deadlock, their locks, and waiting states.
Identify Lock-Ordering Conflicts: A transaction is deadlocked if it has acquired a lock on one resource and is waiting to acquire a lock on another transaction that already holds a lock on the first resource. We need to identify such circular dependencies.
Propose Solution: Suggest changes to ensure transactions acquire locks in a consistent order, or consider additional indexes to reduce contention.
Step-by-Step Analysis

Examine the Deadlock Graph:

The deadlock graph will show transactions as nodes and their waiting states with edges indicating lock requests.
For example:
CODE
T1: X(A) -> T2
T2: X(B) -> T1

Identify Lock-Ordering Conflicts:

In the example above, both transactions T1 and T2 are waiting on each other.
The lock-ordering conflict is that T1 holds an exclusive (X) lock on resource A and wants to acquire an exclusive (X) lock on resource B, while T2 has the reverse situation.

Propose Changes:

Lock Order Consistency: Ensure transactions always request locks in a consistent order, for example, by defining a global lock order.
SQL
-- Example of setting lock order globally
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
Indexing Strategy: Add indexes to reduce the number of locks required. For instance, if T1 and T2 frequently access resources by a common key, consider adding an index on that key.
SQL
CREATE INDEX idx_common_key ON table_name(common_key_column);

#### Closing Summary
+ Common Mistakes/Follow-Ups
Summary: By analyzing the deadlock graph and identifying the lock-ordering conflict, we can propose changes to ensure transactions acquire locks in a consistent order or reduce contention through better indexing.

#### Common Mistakes
Failing to recognize lock-order conflicts as the root cause of deadlocks.
Overlooking transaction isolation levels and their impact on locking behavior.
Not considering the trade-offs between index creation, which can increase write performance but also slow down writes due to additional I/O.
Additional Considerations
Transaction Monitoring: Implement monitoring tools to detect and resolve deadlocks early in production environments.
Database Configuration: Adjust database configuration parameters such as lock_timeout or deadlock_timeout to manage deadlock scenarios more gracefully.

