---
type: "practice"
difficulty: "Intermediate"
---

# How does isolation level impact the correctness of a 'get-or-create' upsert pattern across concurrent requests? Compare INSERT ON CONFLICT versus SELECT-then-INSERT.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Lost Updates and Write Skew](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Lost_Updates_and_Write_Skew.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a "get-or-create" upsert pattern in a concurrent environment, isolation levels play a critical role in ensuring data correctness and consistency. This problem involves understanding how different isolation levels affect transactional behavior—specifically, comparing the INSERT ON CONFLICT approach to the SELECT-then-INSERT pattern.

First, let's define the key concepts:

Isolation Levels: These control the degree of isolation between concurrent transactions.
Get-or-create Upsert Pattern: This is a common idiom where you attempt to insert or update an existing record based on its unique identifier(s).
INSERT ON CONFLICT: A SQL feature that allows you to specify what happens when a conflict (like a duplicate key violation) occurs during an INSERT operation.
SELECT-then-INSERT: This involves first selecting the record and then, if it doesn't exist, performing an INSERT.
TL;DR

Higher isolation levels like RC (Repeatable Read) and RR (Read Stability) provide stronger guarantees but may introduce higher overhead due to locking mechanisms. The INSERT ON CONFLICT approach is generally more efficient in handling concurrency, as it avoids the need for explicit SELECT-INSERT operations.

#### Triage
What Isolation Levels are Being Probed?

The question seeks to understand how different isolation levels impact the correctness of a "get-or-create" pattern when multiple concurrent requests interact with a database. The specific focus is on comparing two common strategies:

INSERT ON CONFLICT: A single transaction that handles both select and insert operations.
SELECT-then-INSERT: A two-step process where you first check for the existence of a record before performing an insert.

#### Assumptions
We assume a shared database with concurrent requests accessing and modifying the same records.
The "get-or-create" pattern is used to ensure that multiple clients can safely update or create records without overwriting each other's data.
We are comparing different isolation levels such as Read Uncommitted (RU), Read Committed (RC), Repeatable Read (RR), and Serializable.

#### First-principles Solution
Isolation Levels Explained
Read Uncommitted (RU): Dirty reads are allowed, meaning a transaction can read uncommitted data from another concurrent transaction.
Read Committed (RC): Ensures that transactions can only read committed data, but may still experience non-repeatable reads and phantom reads.
Repeatable Read (RR): Provides repeatable reads by locking rows during the transaction to prevent others from modifying them until the transaction is complete.
Serializable: Ensures all transactions are serialized, effectively preventing any concurrency issues.
INSERT ON CONFLICT

This approach uses a single transaction to handle both steps:

SQL
INSERT INTO my_table (id, value) VALUES ('unique_id', 'value')
ON CONFLICT (id) DO UPDATE SET value = EXCLUDED.value;
This ensures atomicity and can avoid deadlocks since the entire operation is within one transaction.
SELECT-then-INSERT

This involves two steps:

SELECT: Check if the record exists.
INSERT: If it doesn’t, insert a new record.
Comparative Analysis
Read Uncommitted (RU)

#### Pros
Minimal locking overhead, fast.

#### Cons
May read uncommitted data, leading to potential correctness issues in a concurrent environment.
Read Committed (RC)

#### Pros
Prevents dirty reads and ensures transactions only see committed changes.

#### Cons
Non-repeatable reads may occur. In the "get-or-create" pattern, this can lead to race conditions where multiple transactions might insert the same record if they all read and then insert simultaneously.
Repeatable Read (RR)

#### Pros
Ensures repeatable reads by locking rows during a transaction.

#### Cons
High overhead due to row-level locks. May introduce blocking and deadlocks, particularly in high-concurrency scenarios.
Serializable

#### Pros
Absolute consistency; no concurrency issues.

#### Cons
Highest overhead due to full table scans and exclusive locks on every record. Can lead to significant performance degradation under heavy load.

#### Example
SELECT-then-INSERT with RC Isolation Level

Suppose we have two concurrent transactions:

Transaction A checks if id='unique_id' exists.
If it doesn’t, it inserts the record.
At the same time, Transaction B does the exact same steps.

Without proper locking or isolation, both transactions might read a snapshot of the database where neither has been inserted yet and insert the same record, leading to duplicates.

#### Example
INSERT ON CONFLICT with RC Isolation Level

The INSERT ON CONFLICT approach:

Both transactions start.
Each checks if id='unique_id' exists.
If it does, no action is taken (or an update occurs).
No race conditions arise because the conflict detection happens within a single transaction.

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Common Mistakes
Forgetting to account for concurrency issues, especially in lower isolation levels like RC.

#### Follow-ups
Discussing how these patterns can be optimized further with proper indexing or using optimistic locking mechanisms. Consider the trade-offs between performance and consistency.

#### Conclusion

In summary, while SELECT-then-INSERT can work well under certain isolation levels (like RC), it carries risks of race conditions in high-concurrency environments. The INSERT ON CONFLICT approach generally handles concurrency more gracefully by keeping all steps within a single transaction, ensuring correctness without the need for explicit SELECT operations.

**Code:**

```sql
-- Example INSERT ON CONFLICT statement
INSERT INTO my_table (id, value) VALUES ('unique_id', 'value')
ON CONFLICT (id) DO UPDATE SET value = EXCLUDED.value;
```

**Flow:**

1. Record Exists
2. Record Does Not Exist
3. Record Exists
4. Record Does Not Exist
5. Transaction A
6. Check for Record
7. Do Nothing
8. Insert Record
9. Transaction B
10. Check for Record
11. Do Nothing
12. Insert Record

