---
type: "practice"
difficulty: "Deep"
---

# Why is CREATE INDEX CONCURRENTLY not allowed inside a transaction block, and how does that constrain your migration tool's design?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Concurrent Index Builds](<../../../../01_Databases_and_SQL/Database_Operations/Migrations/Concurrent_Index_Builds/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why CREATE INDEX CONCURRENTLY cannot be used inside a transaction block, we need to explore PostgreSQL’s concurrency management, transaction semantics, and index creation processes. This understanding will help us appreciate the constraints it imposes on database migration tools.

#### Triage
The Question and Probing

The question delves into the intricacies of PostgreSQL's indexing capabilities within transaction blocks. Specifically, it asks why CREATE INDEX CONCURRENTLY is not allowed inside a transaction block and how this affects the design of a migration tool. This reveals several key aspects:

Concurrency Control: Understanding how transactions handle concurrent operations.
Indexing Semantics: The differences between normal index creation and CREATE INDEX CONCURRENTLY.
Migration Tool Design: How these constraints influence the architecture and implementation of migration tools.

#### First-Principles Solution
Concurrency Control in PostgreSQL

PostgreSQL employs a sophisticated locking mechanism to ensure data integrity during concurrent operations. Transactions are atomic, ensuring that either all changes within a transaction block occur or none do. However, this does not always align with the requirements for index creation, especially when using CREATE INDEX CONCURRENTLY.

Indexing Semantics: Normal vs. Concurrency
Normal Index Creation: This operation is synchronous and locks tables to prevent read/write operations during the index build.
CREATE INDEX CONCURRENTLY: This command allows table access while building the index, making it more concurrent but introducing complexity in transaction management.

#### Cons
traints on Transaction Blocks

When an INDEX CONCURRENTLY statement is executed within a transaction block:

The transaction must hold locks long enough to ensure that the index creation process can proceed without interruptions.
If another transaction modifies data during this time, the concurrently building index may become inconsistent or require rollback/rebuild.
Implications for Migration Tools

This limitation impacts migration tools in several ways:

Scheduling Indexes: Migrations often need to build indexes after table population but before application deployment. This requires careful planning and possibly breaking migrations into smaller steps.
Rollback Strategy: If an index creation fails due to concurrent modifications, rollback strategies must account for partial transactions.

#### Closing Summary

CREATE INDEX CONCURRENTLY is not allowed inside a transaction block because of the complexity in managing concurrency and ensuring data consistency. This constraint affects how migration tools are designed, requiring careful planning and potentially breaking migrations into smaller steps to accommodate index creation outside transaction blocks.

#### Common Mistakes
and Follow-Ups
Forgetting Transaction Locks: Assuming that all operations within a transaction will complete without considering the impact on other transactions.
Ignoring Concurrency Issues: Not recognizing how concurrent modifications can affect index builds, leading to inconsistent states or failed migrations.
Overlooking Rollback Scenarios: Failing to plan for and handle situations where index creation fails due to concurrent transactions.

**Code:**

```sql
-- Example of a typical migration step that avoids using CREATE INDEX CONCURRENTLY inside a transaction block

BEGIN;
INSERT INTO users (id, name) VALUES (1, 'Alice');
COMMIT;

BEGIN;
CREATE TABLE IF NOT EXISTS users_indexed AS SELECT * FROM users; -- This is just an example of data replication
-- At this point, the table may be used for queries while index is being created concurrently
CREATE INDEX CONCURRENTLY idx_users_name ON users_indexed (name);
COMMIT;
```

**Flow:**

1. Concurrency Management
2. Concurrency Control
3. Yes
4. No
5. Yes
6. No
7. Start Migration
8. Is Indexing Required?
9. Populate Tables
10. Check for ConcurrentModifications
11. Create Index Normally
12. Wait or Retry with RollbackPlan
13. Finish Migration
14. Handle Partial Operations

