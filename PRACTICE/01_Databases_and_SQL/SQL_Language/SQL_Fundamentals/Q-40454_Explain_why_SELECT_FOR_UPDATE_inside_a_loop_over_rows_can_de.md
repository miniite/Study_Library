---
type: "practice"
difficulty: "Deep"
---

# Explain why SELECT ... FOR UPDATE inside a loop over rows can deadlock when different sessions lock rows in different orders, and how ordering the SELECT (e.g. ORDER BY id) plus SKIP LOCKED avoids it.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ SKIP LOCKED Job Queues](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/SKIP_LOCKED_Job_Queues.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When using SELECT ... FOR UPDATE inside a loop over rows, you can encounter deadlocks due to different sessions locking rows in varying orders. This is because each session may lock the rows in its own order based on the criteria used in the ORDER BY, leading to a situation where two or more transactions block each other indefinitely. To avoid this issue, ordering the SELECT statement with ORDER BY id and adding a SKIP LOCKED clause can prevent deadlocks by ensuring that only unlocked rows are processed.

#### Triage
Understanding the Problem

The question is about understanding how FOR UPDATE locks and the potential for deadlock when processing rows in a loop. The key points to consider are:

Locking Mechanism: Rows are locked using FOR UPDATE.
Session Order: Different sessions may lock rows in different orders.
Deadlock Prevention: Using ORDER BY id and SKIP LOCKED.

#### First-Principles Solution

Let’s break down the problem from first principles:

Row Locking with FOR UPDATE:

When a session executes SELECT ... FOR UPDATE, it locks the matched rows.
This lock is held until the transaction completes, ensuring row-level consistency.

Session-Specific Ordering:

In a loop over rows, each session may start processing from different points or in different orders.
For example, if Session A and Session B both use ORDER BY id but start at different positions, they might lock different sets of rows in an order that prevents them from completing.

Deadlock Scenario:

Suppose Session A locks row 1 with SELECT ... FOR UPDATE, then moves to row 2.
Meanwhile, Session B processes the same set of rows but starts at a later point and needs to lock row 2 first before it can proceed to row 1.

Avoiding Deadlocks:

By ordering the SELECT statement with ORDER BY id, you ensure that both sessions follow the same order.
Adding SKIP LOCKED ensures that only unlocked rows are processed, preventing unnecessary waits and potential deadlocks.
Full Solution

To address this issue effectively, implement the following strategy:

Order the SELECT Statement:

SQL
SELECT * FROM table_name ORDER BY id;

Use FOR UPDATE with SKIP LOCKED:

SQL
SELECT * FROM table_name ORDER BY id FOR UPDATE OF column_name SKIP LOCKED;

Loop and Process Rows:

Loop through the results, processing each row.
Ensure that rows are processed in a consistent order.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By ordering the SELECT statement with ORDER BY id and using FOR UPDATE OF column_name SKIP LOCKED, you can avoid deadlock scenarios caused by session-specific locking orders. The key takeaways are:

Ensuring consistent processing order.
Avoiding unnecessary waits on locked rows.

#### Common mistakes
include:

Failing to understand the implications of different session orders.
Not considering the impact of SKIP LOCKED in preventing deadlocks.

#### Follow-up
questions might include:

How does this solution handle concurrent transactions?
Are there other strategies for managing row locks and preventing deadlocks?

**Code:**

```sql
-- Example SQL query with ORDER BY id and FOR UPDATE SKIP LOCKED
SELECT * FROM table_name ORDER BY id FOR UPDATE OF column_name SKIP LOCKED;
```

