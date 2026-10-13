---
type: "practice"
difficulty: "Expert"
---

# You see InnoDB history list length growing into millions during a nightly batch. Explain the root cause, the production symptoms it will produce, and how you'd remediate without killing the batch outright.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ B-tree Concurrency and Latches](<../../../../01_Databases_and_SQL/Database_Internals/Storage_Engines/Tiered_Storage_in_LSM_Engines/B-tree_Concurrency_and_Latches.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When we observe the InnoDB history list length growing into millions during a nightly batch, this usually points to a performance issue with Innodb_lock_wait_timeout or innodb_locks_unsafe_for_binlog. The root cause is likely related to deadlocks or inefficient locking mechanisms. Here’s an overview of what might be happening and how we can address the issue.

#### Triaging the Question
Understanding the Core Issue:
InnoDB uses a B-tree structure for its undo logs, which are stored in the history list.
When transactions acquire locks on data that are held by other transactions, deadlocks can occur if not managed properly.
Key Concepts to Consider:
InnoDB_lock_wait_timeout: Determines how long InnoDB will wait before timing out and rolling back a transaction due to lock waits.
innodb_locks_unsafe_for_binlog: Controls whether InnoDB allows unsafe situations that could lead to deadlocks.

#### Assumptions
The batch job is running continuously for several hours, causing many transactions.
There are no explicit deadlock detection mechanisms in place.
Step-by-Step Solution

Identify the Root Cause:

Deadlocks: If multiple transactions are waiting on each other indefinitely, deadlocks can occur and lead to a history list buildup.
Timeouts/Aborts: Transactions may be timing out due to InnoDB_lock_wait_timeout settings.

Production Symptoms:

Growing History List Length: As transactions wait for locks or time out, their entries are added to the history list.
Performance Degradation: The growing number of history entries can slow down the system as it needs to manage more undo information.
Error Messages: You may see errors like Deadlock detected while trying to lock table in the logs.

Remediation Strategy:

Adjust Lock Wait Timeout:
SQL
SET GLOBAL innodb_lock_wait_timeout = 50; -- Adjust this value based on your transaction needs.
Optimize Transaction Management:
Ensure that transactions are as short-lived as possible.
Use innodb_adaptive_hash_index and other optimizations to reduce lock contention.
Deadlock Detection Mechanism:
Implement a custom deadlock detection mechanism using MySQL’s information_schema.INNODB_LOCKS, INNODB_LOCK_Waits, or similar tables.

#### Example
query to detect deadlocks:
SQL
SELECT * FROM information_schema.innodb_lock_waits;
Batch Job Optimization:
Break down the batch job into smaller, more manageable chunks.
Use parallel processing where appropriate.

#### Closing Summary
and Common Mistakes

#### Common Mistakes
Overlooking transaction management issues leading to deadlocks.
Not setting innodb_lock_wait_timeout appropriately.
Failing to monitor and optimize transaction lifespans.

#### Follow-Up
Questions:
What other InnoDB settings can affect locking behavior?
How would you integrate these optimizations into your existing infrastructure?

**Code:**

```sql
-- Example of setting innodb_lock_wait_timeout
SET GLOBAL innodb_lock_wait_timeout = 50;
```

