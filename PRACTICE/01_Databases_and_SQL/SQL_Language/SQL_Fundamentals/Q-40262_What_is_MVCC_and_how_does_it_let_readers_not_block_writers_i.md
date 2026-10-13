---
type: "practice"
difficulty: "Expert"
---

# What is MVCC, and how does it let readers not block writers in databases like PostgreSQL?

**Difficulty:** Expert · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ MVCC Bloat and Vacuum](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/MVCC_Bloat_and_Vacuum.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

MVCC stands for Multi-Version Concurrency Control, a key technology used in database systems to ensure that multiple transactions can run concurrently without interfering with each other. In PostgreSQL, MVCC is particularly effective because it allows readers (SELECT statements) to access data from different points in time, even as writers are modifying the database.

To understand how MVCC works and why it prevents blocking between readers and writers, we need to break down its core concepts: versioning of tuples, use of system catalogs for timestamp management, and the role of transaction IDs (XIDs).

#### Triage

The question is asking us to explain MVCC in detail, specifically focusing on how it enables concurrent read operations without blocking write operations. We'll also discuss the internal mechanisms that PostgreSQL uses to implement MVCC.

#### First-Principles Solution

Versioning of Tuples: In a traditional database, if multiple transactions need to access the same row at different times, one transaction might lock the row until it's done with it, preventing other transactions from accessing or modifying the data. This can lead to blocking and reduce concurrency.

System Catalogs for Timestamp Management: PostgreSQL uses system catalogs like pg_class, pg_attribute, and pg_multixact to keep track of transaction IDs (XIDs). Each tuple in a table has an associated version number, which is essentially the XID at which it was last modified.

Transaction IDs (XIDs): Every transaction starts with a unique XID that increments for each new transaction. This allows the database to determine when one transaction started before another and maintain a consistent view of data over time.

Read-Committed Isolation: When a reader query runs, it sees only the committed versions of tuples. If a writer has modified a row but hasn't committed the changes yet, the read operation will not see those uncommitted changes. This means that readers do not block writers because they are operating on a snapshot of the database as of the start of their transaction.

Writer Operations: Writers can modify data without worrying about blocking readers. A writer only blocks other writers until it commits its transaction, ensuring that all modifications become part of the history before allowing new transactions to read or write.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, MVCC in PostgreSQL works by maintaining multiple versions of tuples, using XIDs for timestamp management, and leveraging isolation levels like Read Committed. This design ensures that readers can access data without blocking writers, significantly improving concurrency.

#### Common mistakes
include misunderstanding the concept that MVCC allows concurrent reads without conflicts, or failing to appreciate how transaction IDs track the history of changes. Interviewers might probe deeper by asking about specific scenarios where MVCC behaves differently (e.g., when a writer starts and then rolls back) or how it interacts with other features like transactions and locks.

Code Blocks
SQL
-- Example of using MVCC in PostgreSQL
SELECT * FROM some_table WHERE id = 1 FOR NO KEY UPDATE;

-- This query will read the latest committed version of the row.

