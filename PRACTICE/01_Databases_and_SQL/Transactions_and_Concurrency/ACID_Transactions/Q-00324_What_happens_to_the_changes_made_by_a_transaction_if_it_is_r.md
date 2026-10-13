---
type: "practice"
difficulty: "Basic"
---

# What happens to the changes made by a transaction if it is rolled back?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Commit Rollback and Savepoints](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Commit_Rollback_and_Savepoints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a database transaction is rolled back, all the changes that were made during that transaction are undone, effectively reverting the database to its state before the transaction began. This behavior ensures consistency and integrity in the database by ensuring that incomplete or erroneous transactions do not alter the data permanently.

To understand this concept more deeply, let's break down what a transaction is, how it works, and what happens during a rollback.

