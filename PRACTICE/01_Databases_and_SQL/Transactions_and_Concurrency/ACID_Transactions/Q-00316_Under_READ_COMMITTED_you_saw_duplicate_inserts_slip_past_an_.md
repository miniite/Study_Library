---
type: "practice"
difficulty: "Deep"
---

# Under READ COMMITTED you saw duplicate inserts slip past an application-level check-then-insert; how would you make that operation safe?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Isolation Levels in Transactions](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Isolation_Levels_in_Transactions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview
In a database system, ensuring data integrity is crucial, especially when dealing with concurrent transactions. The READ COMMITTED isolation level allows reading uncommitted changes from other transactions, which can lead to inconsistencies like duplicate inserts slipping through checks at the application level. To address this issue, we need to implement stricter transactional controls.

#### Triage
The problem involves ensuring that a check-then-insert operation is safe under READ COMMITTED. We need to understand how different isolation levels handle concurrency and identify mechanisms to prevent race conditions. Key assumptions include understanding the nature of the application-level checks and the database's locking and indexing capabilities.

#### First Principles Solution

Understand READ COMMITTED Isolation Level: In READ COMMITTED, transactions can read data that has been committed by other transactions but not uncommitted changes made in concurrent transactions. This isolation level does not guarantee that a transaction sees all changes made by concurrently executing transactions, which can lead to the issue of duplicate inserts.

Implement Explicit Locking: To prevent race conditions, we can use explicit locking mechanisms within our application logic. For instance, before performing an insert operation, lock the relevant rows or table to ensure no other transaction modifies them until the current transaction commits.

Use Optimistic Concurrency Control (OCC): OCC involves checking for conflicts at commit time rather than during the read phase. We can implement a version number in our records and increment it with each update. If an insert operation finds a mismatch between the expected and actual version numbers, it fails, indicating that another transaction has modified the data since the last check.

Use Pessimistic Concurrency Control: This approach involves acquiring locks before reading or updating data to prevent other transactions from modifying them until the current transaction is complete. This can be achieved using SELECT ... FOR UPDATE statements in SQL, which lock rows for a given transaction.

Implement Application-Level Retry Logic: In cases where explicit locking or OCC fails due to contention, we can implement retry logic within our application. If an insert operation fails with a unique constraint violation, we can re-run the check-then-insert process after a short delay.

Optimize Indexes and Query Performance: Ensuring that indexes are properly optimized can reduce contention on tables during concurrent transactions, thereby minimizing the likelihood of race conditions.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

The key to making READ COMMITTED safe for check-then-insert operations is understanding how different concurrency control mechanisms can be applied. Explicit locking and optimistic concurrency control are effective strategies but require careful implementation.
A common mistake is relying solely on application-level checks without considering database-level controls, which can lead to overlooked concurrency issues.

#### Follow-ups
might include discussing the trade-offs between different isolation levels, performance implications of locking, and how these solutions scale with increased transactional load.

#### Closing Summary
By understanding the underlying mechanisms and implementing appropriate concurrency control techniques, we can ensure that check-then-insert operations are safe under READ COMMITTED. Careful consideration of both database and application-level strategies is essential to maintaining data integrity in concurrent environments.

