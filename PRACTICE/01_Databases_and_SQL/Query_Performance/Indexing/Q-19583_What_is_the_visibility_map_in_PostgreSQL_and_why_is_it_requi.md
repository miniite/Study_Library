---
type: "practice"
difficulty: "Expert"
---

# What is the visibility map in PostgreSQL and why is it required for an index-only scan?

**Difficulty:** Expert · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Index-only Scans](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index-only_Scans.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The visibility map in PostgreSQL is a bit-based bitmap that tracks which tuples (rows) within a relation are visible to all transactions, meaning they can be returned without requiring a WAL read. It's essential for enabling index-only scans efficiently by reducing the need for additional disk I/O operations.

To understand why the visibility map is required, we first need to delve into how PostgreSQL manages transactional consistency and its storage of data. The visibility map is used in conjunction with index-only scans to ensure that only up-to-date tuples are returned from a query, thereby optimizing read performance without compromising correctness.

#### Triage

The question is asking about the concept and necessity of the visibility map in PostgreSQL, particularly its role in supporting index-only scans. We need to explain what the visibility map is, how it works, and why it's crucial for efficient data retrieval. Assumptions include understanding basic SQL indexing concepts and transactional storage mechanisms.

#### First-Principles Solution
What is a Visibility Map?

A visibility map is essentially a bitmap where each bit corresponds to a specific tuple in the table. A 1 indicates that the tuple is visible (i.e., not deleted or updated by any past transactions), while a 0 means it's invisible and needs additional checks.

Why Is It Required for Index-Only Scans?

Index-only scans are a method of reading data directly from an index, avoiding the need to fetch actual table rows. This is highly efficient but requires ensuring that only up-to-date tuples are returned. The visibility map helps achieve this by pre-filtering which tuples can be considered visible based on their transaction state.

Transaction Management: PostgreSQL uses a multi-version concurrency control (MVCC) model, where each row version is timestamped and marked as visible or invisible based on the transaction's timestamp.
Visibility Check: When an index-only scan occurs, the visibility map allows the database to quickly identify which rows are safe to return without needing to perform additional checks for every tuple.
Detailed Workflow
Build Visibility Map: During a vacuum operation (garbage collection), PostgreSQL updates the visibility map based on transaction timestamps.
Index-Only Scan Execution: When an index-only scan is performed, the database uses the visibility map to quickly identify visible tuples from the index. Only these visible tuples are then fetched directly from their corresponding pages in the table, bypassing any WAL reads.
Benefits
Performance Optimization: Reduces I/O operations by avoiding unnecessary checks.
Data Consistency: Ensures that only up-to-date data is returned, maintaining ACID properties without additional overhead.

#### Closing Summary
+ Common Mistakes/Follow-Ups

The visibility map in PostgreSQL plays a crucial role in enabling efficient index-only scans. By pre-filtering which tuples are visible based on their transaction state, it minimizes the need for additional I/O operations and ensures data consistency. Common mistakes include underestimating its importance or confusing it with other mechanisms like WAL (Write-Ahead Logging).

#### Follow-ups
might involve discussing alternative approaches to achieving similar performance optimizations or exploring edge cases where the visibility map might not be as effective.

**Code:**

```sql
-- Example of a vacuum operation that updates the visibility map
VACUUM ANALYZE my_table;
```

**Flow:**

1. Yes
2. No
3. Start
4. Vacuum Operation UpdatesVisibility Map
5. Transaction Starts
6. Is Tuple Visible?
7. Index-Only Scan Reads DirectlyFrom Index
8. Additional Check Required
9. VACUUM is Needed to UpdateVisibility Map

