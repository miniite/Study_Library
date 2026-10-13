---
type: "practice"
difficulty: "Deep"
---

# Explain why NOLOCK can return duplicate or missing rows even on committed data, and why it is not a safe default for reporting.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Read Committed Locking Anomalies](<../../../../TOOLS/SQL_Server/SQL_Server_Platform/Read_Committed_Locking_Anomalies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When using NOLOCK in SQL Server queries, you might encounter issues like duplicate or missing rows because NOLOCK allows dirty reads, bypassing the database transaction isolation levels that normally prevent such problems. This can lead to inconsistent results and unexpected data states.

In essence, the question is probing your understanding of transaction isolation levels, specifically focusing on how NOLOCK impacts read consistency in SQL Server. Let's break it down step by step:

Transaction Isolation Levels: SQL Server supports four isolation levels: Read Uncommitted, Read Committed, Repeatable Read, and Serializable.
NOLOCK Keyword: The NOLOCK hint instructs SQL Server to perform a dirty read, which means the query can see uncommitted data in addition to committed data.
Why NOLOCK Can Return Duplicate or Missing Rows:
Duplicate Rows: When multiple transactions are running concurrently and using NOLOCK, they might all attempt to read from the same row at the same time but in different states. For instance, one transaction might have just updated a row that another is reading. This can cause the second transaction to see an outdated value or even a ghost image of the row if it's being modified by another transaction.
Missing Rows: Similarly, NOLOCK can result in missing rows because other transactions might be rolling back changes that were not yet committed when your query ran. If you read from such a transaction while it’s still active but hasn’t committed its changes, the row may appear to be gone.

To illustrate, consider this scenario:

Transaction A updates a row with a value of 10.
Transaction B uses NOLOCK and reads the same row, seeing the old value (let's say 5).
Before Transaction A commits, Transaction C also uses NOLOCK and reads the row again but sees the new value of 10 because it bypassed the uncommitted change.

This can lead to situations where your reporting data is inconsistent or inaccurate, which is why NOLOCK should not be a default setting for reporting queries.

**Code:**

```sql
-- Example Query with NOLOCK Hint
SELECT * FROM Orders WITH (NOLOCK)
```

