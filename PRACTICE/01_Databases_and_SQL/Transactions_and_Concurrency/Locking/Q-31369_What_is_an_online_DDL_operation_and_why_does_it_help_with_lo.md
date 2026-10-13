---
type: "practice"
difficulty: "Deep"
---

# What is an online DDL operation and why does it help with locking during schema changes?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ MVCC](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Locking/MVCC.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Online Data Definition Language (DDL) operations are a class of database operations designed to make schema changes without interrupting ongoing transactions or reads. The primary benefit is maintaining system availability and performance during critical maintenance tasks, such as adding new columns, creating indexes, or dropping tables.

#### Triage

Question: What is an online DDL operation and why does it help with locking during schema changes?
Key Aspects: Understand what DDL operations are, the concept of "online" in this context, and how it impacts locking.

#### Assumptions
Assume a basic understanding of database transactions and concurrency control.

Step-by-step Solution:

Understanding DDL Operations:

DDL commands include CREATE, ALTER, and DROP statements used to define or modify the structure of databases or tables.
In traditional systems, executing such operations can lead to full table locks, blocking all concurrent transactions until the schema change is completed.

What "Online" Means:

An online DDL operation allows you to perform these changes without taking the entire database offline or significantly impacting ongoing transactions.
This involves using advanced locking and concurrency control mechanisms that allow some parts of the system to continue operating while others are changing.

Concurrency Control Mechanisms:

MVCC (Multiversion Concurrency Control): Used by databases like PostgreSQL, it allows multiple versions of data to coexist.
SQL
-- Example in PostgreSQL using MVCC
CREATE TABLE new_table AS SELECT * FROM old_table WHERE true;  -- Creates a copy without blocking
Row-level Locking: Ensures that only specific rows are locked during the DDL operation, allowing other transactions to proceed.
Snapshot Isolation: Allows queries to see a consistent snapshot of the database state at the time the query was issued.

Impact on Locking:

By using these mechanisms, online DDL operations minimize the scope and duration of locks.
This prevents long-term table-level locks that would otherwise block all read and write operations during schema changes.

Use Cases:

Enabling ongoing data processing without interruptions.
Simplifying maintenance tasks by allowing them to be performed at more convenient times.

#### Closing Summary
Online DDL operations leverage advanced locking and concurrency control techniques to perform schema changes with minimal impact on system availability, ensuring that the database remains operational during critical maintenance tasks. Understanding these mechanisms is crucial for designing robust and scalable database systems.

#### Common Mistakes/Follow-ups

Misunderstanding Full Table Locks: Thinking that all DDL operations involve full table locks.
Ignoring Performance Impact: Failing to recognize that even with online DDL, there can still be performance implications during the change.
Overlooking Concurrency Control Techniques: Not understanding how MVCC and row-level locking contribute to maintaining system availability.

**Code:**

```sql
-- Example of an online DDL operation using PostgreSQL's MVCC
CREATE TABLE new_table AS SELECT * FROM old_table WHERE true;  -- Creates a copy without blocking
```

**Flow:**

1. Yes
2. No
3. Start
4. Is DDL operation online?
5. Perform DDL with MVCC
6. Full table lock, blocktransactions
7. Ensure concurrent reads/writecan proceed
8. Complete changes, maintainsystem availability
9. Transactions wait forcompletion
10. End

