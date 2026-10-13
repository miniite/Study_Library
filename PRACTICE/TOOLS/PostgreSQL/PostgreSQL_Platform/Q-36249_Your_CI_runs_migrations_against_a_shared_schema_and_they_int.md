---
type: "practice"
difficulty: "Intermediate"
---

# Your CI runs migrations against a shared schema and they intermittently deadlock during CREATE INDEX; how would you serialize DDL and set lock_timeout so a migration fails fast instead of blocking the whole app?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Deadlocks and Lock Ordering](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Deadlocks_and_Lock_Ordering.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of deadlocks occurring during migrations in PostgreSQL, we need to understand that migrations often run concurrently with application operations, which can lead to conflicts when multiple transactions try to modify the same schema. The goal is to serialize DDL operations and set appropriate lock timeouts so that a migration fails fast rather than blocking other critical processes.

We are probing your understanding of PostgreSQL’s transactional behavior, concurrency control mechanisms, and best practices for managing migrations in production environments. We will assume standard multi-tenant or high-concurrency database scenarios where such issues might arise.

Firstly, let's walk through the problem step-by-step:

Understand the Issue: The migrations are running as part of a CI/CD pipeline and are interfering with each other due to locking conflicts.
Serialize DDL Operations: PostgreSQL does not natively support serializing DDL operations across sessions; however, we can manage the order of execution within a single session using transactions.
Set Lock Timeout: Configure the lock_timeout parameter to ensure that if a transaction cannot acquire necessary locks within a certain timeframe, it will fail gracefully.

Now let's go through each step in detail:

Step 1: Understanding DDL Operations and Concurrency

DDL (Data Definition Language) operations such as CREATE INDEX, ALTER TABLE, etc., are not serializable by default because they modify the schema. When multiple migrations run concurrently, they can lock the same objects and cause deadlocks.

Step 2: Serializing DDL Operations in a Single Session

To serialize DDL operations within a single session, you can use transaction blocks:

SQL
BEGIN;
-- First migration operation
CREATE INDEX idx_table1_column ON table1(column);
COMMIT;

BEGIN;
-- Second migration operation
CREATE INDEX idx_table2_column ON table2(column);
COMMIT;

By wrapping each migration step in a separate transaction block, you ensure that the operations are serialized and do not interfere with each other.

Step 3: Setting Lock Timeout

To handle situations where locks cannot be acquired within a reasonable time, we need to set the lock_timeout parameter. This can be done globally or on a per-session basis:

SQL
-- Globally set lock timeout to 10 seconds (10000 ms)
SET LOCAL lock_timeout = '10000ms';

Alternatively, you can use the LOCK TABLE command to explicitly control locking behavior for specific tables during migrations.

Step 4: Implementing a Custom Locking Mechanism

For more complex scenarios, you might want to implement a custom locking mechanism using PostgreSQL’s advisory locks or row-level locks:

SQL
-- Acquire an advisory lock before starting the migration
SELECT pg_advisory_lock(12345);

BEGIN;
-- Migration operations
CREATE INDEX idx_table_column ON table(column);
COMMIT;

-- Release the lock after completion
SELECT pg_advisory_unlock(12345);
Step 5: Monitoring and Logging

Finally, monitor the application logs to identify deadlocks and failures. Use PostgreSQL’s pg_locks view to inspect locking behavior:

SQL
SELECT * FROM pg_locks WHERE locktype = 'relation';

This will help you understand where locks are being held and why they might be causing issues.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By serializing DDL operations, setting appropriate lock_timeout, and using custom locking mechanisms, we can mitigate the risk of deadlocks during migrations. However, it's crucial to test these changes in a staging environment before rolling them out to production.

#### Common Mistakes

Failing to serialize DDL operations within transactions.
Setting excessively high lock timeouts which could cause performance issues.
Not properly releasing locks after they are no longer needed.

In follow-up questions, you might be asked about the trade-offs between failing fast versus blocking, how to prioritize migrations in a multi-tenant environment, or how to integrate these changes into your CI/CD pipeline.

**Code:**

```sql
-- Example of serializing DDL operations
BEGIN;
CREATE INDEX idx_table_column ON table(column);
COMMIT;

BEGIN;
CREATE INDEX idx_table2_column ON table2(column);
COMMIT;
```

