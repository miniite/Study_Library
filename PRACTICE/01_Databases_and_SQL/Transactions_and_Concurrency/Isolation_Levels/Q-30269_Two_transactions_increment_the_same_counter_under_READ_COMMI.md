---
type: "practice"
difficulty: "Intermediate"
---

# Two transactions increment the same counter under READ COMMITTED and one update is lost. Walk me through why, and three ways to prevent the lost update.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Lost Updates and Write Skew](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Lost_Updates_and_Write_Skew.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a database system operating under the READ COMMITTED isolation level, multiple transactions can read and write data without locking, which means that one transaction might see an uncommitted change made by another transaction. In this scenario, we'll examine why a lost update might occur when two transactions increment the same counter under READ COMMITTED and propose three strategies to prevent it.

#### Triaging the Question

Question Restatement:

Why does a lost update happen with two transactions incrementing the same counter under READ COMMITTED isolation level?
How can we prevent this issue?

Key Assumptions:

The database is using READ COMMITTED isolation.
Two concurrent transactions are attempting to increment the same counter simultaneously.

#### First Principles Solution
Lost Update Under READ COMMITTED

In a database with READ COMMITTED isolation, each transaction reads and writes data without acquiring locks. Here’s what happens in our scenario:

Transaction T1:

Starts.
Reads the current value of the counter (let's say it is 5).
Increments the counter to 6 but doesn’t commit yet.

Transaction T2:

Starts after T1 has read the counter but before T1 commits.
Also reads the counter (seeing the same value, 5) because READ COMMITTED only locks the data for write operations and not during reading.
Increments the counter to 6.

T1 Commits:

T1 then commits its transaction, setting the counter to 6.

T2 Commits:

However, T2 does not see T1’s commit because it was already reading a snapshot of the database state (5) before T1 committed.
T2 also commits, but this time the counter value is still 6.

Thus, in this scenario, both transactions are incrementing the same counter to 6, and one transaction's update is effectively lost. This happens because neither transaction sees the other’s uncommitted changes.

Preventing Lost Updates

To prevent a lost update under READ COMMITTED isolation, we can implement the following strategies:

Use REPEATABLE READ:

By setting the database to use REPEATABLE READ isolation level, both T1 and T2 would lock the row they are reading until their transaction is completed.
When T1 starts its transaction, it locks the counter row, preventing T2 from reading or writing to that row.
If T2 tries to read the same row while T1 has a lock, it will wait for T1 to release the lock.

Optimistic Locking:

Implement optimistic locking by adding a version column (e.g., version or timestamp) in the counter table.
Each transaction reads the current value of this version column along with the counter.
On commit, the transaction checks if the version column still matches what it read earlier; if not, it retries.

Row-Level Locking:

Use explicit row-level locks to ensure that only one transaction can modify a specific row at a time.
For example, in T1 and T2, both could lock the counter row before performing their increment operation, ensuring no other transactions can interfere.

#### Closing Summary

To summarize:

Lost updates occur under READ COMMITTED because transactions do not see each other's uncommitted changes.
Three strategies to prevent this are: switching to REPEATABLE READ isolation level, implementing optimistic locking, and using explicit row-level locks.

Understanding these strategies helps in choosing the right approach based on application requirements and database characteristics.

#### Common Mistakes
Misunderstanding Isolation Levels: Assuming that READ COMMITTED prevents lost updates.
Ignoring Locking Mechanisms: Not realizing that transactions need to be aware of each other's presence through locking mechanisms.
Overlooking Concurrency Control: Failing to recognize the importance of concurrency control in preventing race conditions.

