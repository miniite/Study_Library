---
type: "practice"
difficulty: "Deep"
---

# What are the different isolation levels Oracle supports and how does Oracle differ from the SQL standard on this?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Oracle Isolation and Read Consistency](<../../../../TOOLS/Oracle/Oracle_Platform/Oracle_Isolation_and_Read_Consistency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the different isolation levels supported by Oracle and how they differ from the SQL standard, we need to first grasp the purpose of transaction isolation levels in database management systems (DBMS). These levels determine the level of concurrency control provided by a DBMS, ensuring that concurrent transactions do not interfere with each other's data integrity. In Oracle Database, there are four main isolation levels: READ COMMITTED, READ UNCOMMITTED, SERIALIZABLE, and LOCAL TIMESTAMP.

#### Triage

Oracle supports these isolation levels to manage how transactions interact with shared data in terms of visibility and locking. The SQL standard also defines several isolation levels (READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, SERIALIZABLE), but there are nuanced differences between Oracle's implementation and the standard.

#### First-Principles Solution
READ UNCOMMITTED

This is the lowest level of isolation. In this mode, a transaction can read uncommitted data from other transactions. This means that dirty reads can occur, as a transaction might see data that has been written but not yet committed by another transaction.

SQL
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
READ COMMITTED

This is the default isolation level in Oracle and most SQL-compliant systems. It ensures that transactions only read committed data. A dirty read cannot occur, as uncommitted changes are ignored.

SQL
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
REPEATABLE READ

In this mode, once a transaction has read a row of data (or part of it), the row will not change for that transaction until the transaction is complete. This prevents non-repeatable reads and phantom rows. However, it can lead to higher locking overhead.

SQL
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SERIALIZABLE

The highest isolation level ensures that all operations in a transaction occur as if they were executed serially (one after another), even though multiple transactions might be running concurrently. This is achieved by taking out locks on the data being accessed, which can significantly impact performance due to increased locking and potential deadlocks.

SQL
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
Oracle's LOCAL TIMESTAMP Isolation Level

Oracle also supports a local timestamp isolation level that allows transactions to be isolated based on timestamps. This provides more fine-grained control over transaction visibility, allowing for different levels of isolation within the same transaction.

SQL
SET TRANSACTION ISOLATION LEVEL LOCAL TIMESTAMP FOR CURRENT ROW USING SYSTEM;

#### Closing Summary
+ Common Mistakes/Follow-Ups

Oracle's implementation aligns closely with the SQL standard but offers additional features like LOCAL TIMESTAMP. The choice of isolation level depends on the specific requirements of the application, balancing between data consistency and performance.

#### Common mistakes
include using a higher isolation level than necessary, which can lead to unnecessary locking and performance degradation, or not understanding the implications of different levels when dealing with concurrent transactions.

Depth + Language Match

As a Database Developer with deep understanding (level 2/3), you should be able to explain these concepts clearly and provide context on their practical application in various scenarios. Understanding the trade-offs between isolation levels is crucial for designing efficient database applications.

**Code:**

```sql
-- Example of setting transaction isolation level in Oracle
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
```

