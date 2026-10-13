---
type: "practice"
difficulty: "Deep"
---

# In Oracle you get ORA-08177 'can't serialize access' under SERIALIZABLE. What causes it and how do you design the app to handle it?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Serializable Performance](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/Serializable_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

When working with Oracle's SERIALIZABLE isolation level, you might encounter the ORA-08177 'can't serialize access' error. This happens when two transactions are unable to complete successfully due to conflicting data modifications or locks held by one transaction preventing the other from proceeding in a way that would maintain serializability. To handle this issue effectively, you need to understand the isolation level's constraints and design your application to manage concurrent transactions and potential deadlocks.

#### Triage

The question asks about the ORA-08177 error under SERIALIZABLE isolation in Oracle. The key points to probe are:

Understanding Isolation Levels: What does SERIALIZABLE mean?
ORA-08177 Error: How and why this specific error occurs.
Transaction Design: How can you design your application to handle such errors gracefully?

#### First-Principles Solution
1. Understanding SERIALIZABLE Isolation Level

SERIALIZABLE isolation ensures that transactions are executed in a serial order, meaning they appear as if they were run one at a time rather than concurrently. This prevents non-repeatable reads and phantom reads but can lead to serialization issues.

2. The ORA-08177 Error

ORA-08177 'can't serialize access' is thrown when Oracle cannot achieve the isolation level requested due to conflicting transactions. Specifically, it occurs in one of these scenarios:

Write Conflicts: When a transaction writes to data that another transaction has already locked for writing.
Read Conflicts: When a transaction reads data that another transaction has locked for writing.
3. Designing the Application

To handle ORA-08177, you need to design your application with these considerations:

a) Optimistic vs. Pessimistic Locking

Pessimistic Locking: Explicitly lock records before reading or updating them.
SQL
BEGIN
  FOR rec IN (SELECT * FROM table WHERE id = 1 FOR UPDATE) LOOP;
    -- Perform updates
  END LOOP;
END;
Optimistic Locking: Read a row and check for changes after modification.
SQL
SELECT * INTO var FROM table WHERE id = 1 AND version = 1 FOR UPDATE;
-- Update logic here
UPDATE table SET column = 'new value' WHERE id = 1 AND version = 1;

b) Deadlock Detection and Resolution

Implement a timeout mechanism to detect and resolve deadlocks.
Use DBMS_LOCK package for manual locking and unlocking.
SQL
DECLARE
  l_mode PLS_INTEGER := 0; -- Shared mode
BEGIN
  DBMS_LOCK.allocate_unique('my_lock', 'lock1');
  DBMS_LOCK.request(l_mode, 'other_resource_name');
  -- Perform operations
  DBMS_LOCK.release;
END;

c) Transaction Management

Ensure transactions are short-lived to minimize lock contention.
Use savepoints for partial transaction rollbacks.
4. Example Scenario and Dry-Run

#### Cons
ider two concurrent transactions:

Transaction A: Reads record id = 1 with version 1, updates it.
Transaction B: Simultaneously reads the same record id = 1, tries to update it.

If Transaction B runs first, locks the row, and then Transaction A modifies the data, Oracle will throw ORA-08177 when Transaction A attempts to write its changes because Transaction B has a lock on the row. To resolve this, you can implement pessimistic locking or use savepoints in both transactions.

SQL
-- Transaction A (Pessimistic Locking)
BEGIN
  FOR rec IN (SELECT * FROM table WHERE id = 1 FOR UPDATE) LOOP;
    -- Perform updates
  END LOOP;
END;

-- Transaction B (Optimistic Locking)
DECLARE
  l_version NUMBER := 1; -- Expected version
BEGIN
  SELECT * INTO var FROM table WHERE id = 1 AND version = l_version FOR UPDATE;
  IF SQL%FOUND THEN
    -- Update logic here, increment version number
    UPDATE table SET column = 'new value', version = version + 1 WHERE id = 1;
  ELSE
    RAISE_APPLICATION_ERROR(-20500, 'Transaction failed due to concurrency issues');
  END IF;
END;

#### Closing Summary
and Common Mistakes/Follow-Ups

To avoid ORA-08177, focus on proper transaction design using locking strategies. Pessimistic locking can help manage write conflicts, while optimistic locking ensures data consistency in read-heavy scenarios. Always consider implementing timeout mechanisms to detect deadlocks and handle them gracefully.

#### Common mistakes
include:

Not understanding the isolation level's constraints.
Failing to implement adequate locking mechanisms.
Ignoring concurrency control when designing transactions.

#### Follow-ups
might involve discussing specific use cases, trade-offs between different lock types, or more advanced deadlock detection techniques using Oracle's built-in tools like DBMS_LOCK.

**Comparison:**

```
Column Name	Data Type
id	NUMBER
column	VARCHAR2
version	NUMBER
```

**Code:**

```sql
BEGIN
  FOR rec IN (SELECT * FROM table WHERE id = 1 FOR UPDATE) LOOP;
    -- Perform updates
  END LOOP;
END;

DECLARE
  l_version NUMBER := 1; -- Expected version
BEGIN
  SELECT * INTO var FROM table WHERE id = 1 AND version = l_version FOR UPDATE;
  IF SQL%FOUND THEN
    -- Update logic here, increment version number
    UPDATE table SET column = 'new value', version = version + 1 WHERE id = 1;
  ELSE
    RAISE_APPLICATION_ERROR(-20500, 'Transaction failed due to concurrency issues');
  END IF;
END;
```

**Example input / output:**

```text
Input:
- Transaction A reads record `id = 1` with version 1, updates it.
- Simultaneous read of the same record by Transaction B.

Output:
- ORA-08177 'can't serialize access' error if transaction A cannot proceed due to lock held by transaction B.
```

**Data:**

```sql
CREATE TABLE table (
  id NUMBER,
  column VARCHAR2(100),
  version NUMBER
);
```

**Flow:**

1. Transaction A Reads andUpdates
2. Reads Record
3. Locks Record
4. Transaction B SimultaneouslyReads
5. Reads Same Record
6. Locks Record for Write
7. Transaction A Tries to Update
8. Checks Lock
9. Transaction B Holds Lock,ORA-08177

