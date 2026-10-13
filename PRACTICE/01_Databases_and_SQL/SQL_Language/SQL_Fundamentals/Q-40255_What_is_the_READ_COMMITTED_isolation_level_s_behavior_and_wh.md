---
type: "practice"
difficulty: "Deep"
---

# What is the READ COMMITTED isolation level's behavior, and why is it the common default?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Isolation Anomalies](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Isolation_Anomalies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: The READ COMMITTED isolation level ensures that a transaction sees only data committed before it started. This is the most common default because it strikes a balance between performance and consistency, providing reasonable protection against dirty reads while minimizing locking overhead.

The question you're asking delves into one of the core concepts in database transaction management: the READ COMMITTED isolation level. In this isolation level, a transaction sees only data that was committed before the transaction began. Let's break down why this is both practical and commonly used as the default.

#### Triage
What the Question Is Asking
Behavior of READ COMMITTED: You need to describe how transactions behave under READ COMMITTED.
Why It’s Commonly Defaulted: Understand the reasoning behind its prevalence, especially in terms of performance and consistency trade-offs.
Step-by-Step First Principles Solution
1. Understanding READ COMMITTED

READ COMMITTED is a database isolation level that guarantees that when you start reading data (a transaction), you can only see changes made by transactions committed before your current transaction started. This means:

No Dirty Reads: You won't read uncommitted data from other transactions.
Some Level of Non-Repeatable Reads and Phantom Reads: Because the isolation level doesn’t lock rows or tables, a transaction might see different values for the same row during multiple reads (non-repeatable read) or new rows that appear after your first query (phantom read).
2. Performance vs Consistency Trade-offs

READ COMMITTED is often chosen as the default because:

Minimal Locking Overhead: Transactions do not lock data unless they are updating it, which can speed up transactional operations and reduce contention.
Readers Don’t Block Writers: This means that while a writer locks a resource, readers can still access other resources. This minimizes blocking and deadlocks.
3. Practical Example

#### Cons
ider two transactions:

Transaction A reads the balance of an account at time t0.
Before Transaction A finishes, Transaction B writes to that account’s balance.
When Transaction A commits after reading (at t1), it sees the updated balance because READ COMMITTED ensures only committed data.
4. Edge Cases and Failure Modes
Non-repeatable Read: If Transaction A reads an account twice, it might get different values due to updates by other transactions.
Phantom Reads: New rows inserted by another transaction after the initial read may not be visible in subsequent queries.

#### Closing Summary
+ Common Mistakes/Follow-ups

Summary: The READ COMMITTED isolation level is a practical default because it strikes a balance between performance and consistency. It minimizes locking overhead while still providing protection against dirty reads, albeit with some trade-offs like non-repeatable reads and phantom reads.

#### Common Mistakes

Assuming READ COMMITTED prevents all types of concurrency issues.
Overlooking the need for application-level logic to handle non-repeatable reads and phantom reads when using this isolation level.

#### Follow-ups
How does it compare with other isolation levels (SERIALIZABLE, REPEATABLE READ)?
What are some scenarios where READ COMMITTED might not be suitable?
How do you handle non-repeatable read and phantom read issues in an application using READ COMMITTED?

**Code:**

```sql
-- Example of using SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

