---
type: "practice"
difficulty: "Deep"
---

# How does a cursor's FOR UPDATE NOWAIT versus FOR UPDATE WAIT n change behavior when another session holds the row locks?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Ref Cursor Consistency and Limits](<../../../../01_Databases_and_SQL/Database_Objects/Cursors/Ref_Cursor_Consistency_and_Limits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with cursors in SQL, the FOR UPDATE clause is used to lock rows so that they cannot be modified by other transactions until the current transaction completes. The key difference between FOR UPDATE NOWAIT and FOR UPDATE WAIT lies in how they handle row locks held by other sessions.

To understand this, let's break down each option:

FOR UPDATE NOWAIT: If another session holds a lock on the row(s), the command will fail immediately without waiting. This is useful in scenarios where you need to quickly check if a row can be updated and are willing to handle any potential conflicts by retrying the operation.

FOR UPDATE WAIT: The transaction waits until it can acquire the necessary locks. If another session holds the lock, FOR UPDATE WAIT will block and wait for that session to release its lock before proceeding with the current transaction.

The choice between these two options depends on your application's requirements and how you handle concurrency issues.

#### Triaging the Question
What the Question is Asking

This question asks about the behavior of cursors when FOR UPDATE NOWAIT and FOR UPDATE WAIT are used, especially in the context where another session already holds row locks.

#### Assumptions
We assume a multi-session environment.
The transaction using FOR UPDATE has an exclusive lock requirement on certain rows.
Another session is holding one or more of these rows locked with either SELECT FOR UPDATE, UPDATE, or DELETE.

#### First-Principles Solution

Understanding Locks:

When a row is updated with FOR UPDATE, the database engine places an exclusive lock on that row.
If another session has already acquired such a lock, it depends on the chosen option (NOWAIT or WAIT) for how to proceed.

For FOR UPDATE NOWAIT:

The current transaction will attempt to acquire the locks but will immediately return an error if any of the rows are already locked by another session.

#### Example
If Session A executes SELECT * FROM table WHERE id = 1 FOR UPDATE, and before Session B can execute a similar command, Session C starts a new session that also tries to lock row 1 with FOR UPDATE NOWAIT. In this case, the second transaction will fail immediately because row 1 is already locked by Session A.

For FOR UPDATE WAIT:

The current transaction will wait until all necessary locks can be acquired.

#### Example
If Session D tries to lock row 2 with SELECT * FROM table WHERE id = 2 FOR UPDATE WAIT and row 2 is currently being held by Session E, the transaction in Session D will block. It will remain blocked until Session E releases its lock on row 2.

Handling Concurrency:

FOR UPDATE NOWAIT: This option avoids blocking but might result in frequent retries or failures if concurrent transactions are common.
FOR UPDATE WAIT: This ensures that the transaction will proceed once all necessary locks are acquired, which can lead to longer wait times and potential performance issues due to blocking.

#### Closing Summary

In summary, FOR UPDATE NOWAIT is useful when you want to quickly check for row availability without waiting, while FOR UPDATE WAIT ensures that the transaction can acquire all required locks before proceeding. The choice between these options should be made based on your application's needs and how you plan to handle concurrent transactions.

#### Common Mistakes/Follow-Ups
Forgetting to account for the possibility of NOWAIT failing, leading to potential data inconsistencies.
Misunderstanding the blocking behavior of WAIT, which can lead to performance issues if not managed correctly.
Not considering retry logic when using NOWAIT in high-concurrency environments.

**Code:**

```sql
-- Example with FOR UPDATE NOWAIT
SELECT * FROM table WHERE id = 1 FOR UPDATE NOWAIT;

-- Example with FOR UPDATE WAIT
SELECT * FROM table WHERE id = 2 FOR UPDATE WAIT;
```

**Flow:**

1. SELECT * FROM tableWHERE id = 1 FOR UPDATE
2. SELECT * FROM tableWHERE id = 2 FOR UPDATEWAIT
3. SELECT * FROM tableWHERE id = 2 FOR UPDATE
4. Session A
5. Row 1 Locked
6. Session D
7. Blocked until Session EReleases Lock
8. Session E
9. Row 2 Locked

