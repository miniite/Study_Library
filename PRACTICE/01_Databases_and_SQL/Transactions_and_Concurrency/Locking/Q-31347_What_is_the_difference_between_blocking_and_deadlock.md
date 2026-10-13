---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between blocking and deadlock?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Row Locks Two-phase Locking and Lock Footprint](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Row_Locks_Two-phase_Locking_and_Lock_Footprint.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database systems, particularly in relational databases with concurrent access, understanding how data is accessed and modified is crucial to ensure consistency and performance. Blocking and deadlocks are two common issues that can arise during concurrency control. Here’s a brief overview before diving into the details:

Blocking occurs when a transaction waits for another transaction to release a lock on a resource it needs. Deadlock, on the other hand, happens when two or more transactions are waiting for each other to release locks, resulting in an infinite wait.

To understand these concepts better, let's break them down from first principles and explore how they differ.

Blocking vs Deadlock: What Are We Probing?

Blocking:

Blocks one transaction until another transaction releases a lock.
Often temporary; the blocked transaction will resume once it can acquire the necessary locks.

Deadlock:

Occurs when multiple transactions are waiting for each other to release resources, leading to an indefinite wait.

#### Triaging the Question

The interviewer is likely probing your understanding of concurrency control mechanisms and their implications in database systems. We need to explain both concepts clearly, state assumptions, and explore the scenarios where they occur.

#### Assumptions
Transactions are modifying data that requires locks.
Locks can be shared (read) or exclusive (write).
Multiple transactions operate concurrently.

#### First-Principles Solution

Blocking:

Occurs when a transaction ( T1 ) is waiting for another transaction ( T2 ) to release an exclusive lock on data it needs.
For example, if ( T1 ) wants to write to row A and ( T2 ) has an exclusive lock on the same row, ( T1 ) will be blocked until ( T2 ) releases its lock.

Deadlock:

Happens when two or more transactions are waiting for each other to release resources.
Each transaction holds a resource and waits for another transaction that also holds a resource it needs.

Let's illustrate this with an example:

Suppose we have two transactions ( T1 ) and ( T2 ):
( T1 ): Acquires exclusive lock on row A (writes to A).
( T2 ): Acquires shared lock on row B (reads from B) and then needs an exclusive lock on row A.

If ( T1 ) holds the exclusive lock on row A, ( T2 ) will be blocked until ( T1 ) releases its lock. However, if both ( T1 ) and ( T2 ) simultaneously try to acquire locks in such a way that each waits for the other (e.g., ( T1 ) needs B and ( T2 ) needs A), they enter a deadlock.

Worked Example

Assume:

Row A: T1 writes
Row B: T2 reads, then wants exclusive lock on A

Transaction T1:

Acquires exclusive lock on row A.
Holds the lock while writing to A.

Transaction T2:

Acquires shared lock on row B (reading from B).
Requests an exclusive lock on row A (write).

Since ( T2 ) is already holding a shared lock, it will be blocked until ( T1 ) releases its exclusive lock on A. But if at the same time:

Transaction T1:
Requests an exclusive lock on row B.
Is blocked because ( T2 ) holds a shared lock on B.

In this scenario, both transactions are waiting for each other to release locks, resulting in a deadlock.

#### Closing Summary

Blocking and deadlocks are critical concepts in database concurrency control. Blocking is typically temporary and resolved when the blocking transaction releases its locks, whereas deadlocks can be more problematic as they result in indefinite waits if not handled properly.

#### Common mistakes
include failing to detect or resolve deadlocks promptly, leading to system performance degradation. It's important to implement mechanisms like lock timeouts, deadlock detection algorithms (like the Banker’s algorithm), and optimistic concurrency control strategies to mitigate these issues.

#### Common Mistakes/Follow-Ups
Mistake: Not recognizing a situation that could lead to a deadlock.

#### Follow-Up
Discussing how to detect and resolve deadlocks using mechanisms like serialization sets or by periodically checking for cycles in the lock graph.

**Comparison:**

```
Locks row A exclusively
Reads and locks A shared
Transaction T1
Blocking Transaction
Transaction T2
Deadlock Transaction
Trans T1 waits for T2 to releaselock on A
Trans T2 waits for T1 to releaselock on A
```

**Code:**

```

-- Example of blocking scenario BEGIN TRANSACTION; LOCK TABLES table1 WRITE; -- T1 acquires exclusive lock on A

-- Simulating delay SELECT sleep(5);

ROLLBACK; -- T1 releases its lock, allowing T2 to proceed

CODE
Copy

```sql
-- Example of deadlock scenario
BEGIN TRANSACTION;
SELECT * FROM table1 WHERE id = 1 FOR UPDATE; -- T2 acquires shared lock on A

-- Simulating delay
SELECT sleep(5);

UPDATE table1 SET value = 'new' WHERE id = 1; -- T1 requests exclusive lock, blocked by T2's shared lock
```

**Example input / output:**

```

-- Input BEGIN TRANSACTION; LOCK TABLES table1 WRITE; -- T1 acquires exclusive lock on A

-- Simulating delay SELECT sleep(5);

ROLLBACK; -- T1 releases its lock, allowing T2 to proceed

-- Output (expected)

CODE
Copy

```sql
-- Input
BEGIN TRANSACTION;
SELECT * FROM table1 WHERE id = 1 FOR UPDATE; -- T2 acquires shared lock on A

-- Simulating delay
SELECT sleep(5);

UPDATE table1 SET value = 'new' WHERE id = 1; -- T1 requests exclusive lock, blocked by T2's shared lock

-- Output (expected)
```

**Data:**

```json
{
  "table1": {
    "id": [1],
    "value": ["original"]
  }
}
```

**Flow:**

1. Locks row A exclusively
2. Reads and locks A shared
3. Transaction T1
4. Blocking
5. Transaction T2
6. Deadlock
7. Trans T1 waits for T2 to releaselock on A
8. Trans T2 waits for T1 to releaselock on A

