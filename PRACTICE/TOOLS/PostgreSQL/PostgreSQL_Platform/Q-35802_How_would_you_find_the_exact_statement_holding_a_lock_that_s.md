---
type: "practice"
difficulty: "Intermediate"
---

# How would you find the exact statement holding a lock that's blocking a migration, using pg_locks joined to pg_stat_activity, and then decide between waiting, canceling, or killing the blocker?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Deadlocks and Lock Ordering](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Deadlocks_and_Lock_Ordering.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

SQL
-- Step 1: Find the process ID (PID) of the blocking query
WITH blocking_query AS (
    SELECT
        l.locktype,
        l.relation::regclass AS relation, -- Convert OID to table name
        l.githubusercontent.com/username/repo/commit/cb45f7d1b8a3c0f6d46096a2b4d9f0a922e22e9d/raw/pg_locks.sql#L18
        s.query,
        s.pid AS blocking_pid,
        s.state::text,
        l.granted -- Lock is held if granted = 't'
    FROM
        pg_catalog.pg_locks l
    JOIN
        pg_stat_activity s ON l.pid = s.pid
    WHERE
        l.locktype IN ('relation', 'transactionid') AND  -- Only consider relation and transaction locks
        NOT l.granted  -- The lock is not granted (held by another transaction)
)

-- Step 2: Identify the query causing the blocking
SELECT
    bq.blocking_pid,
    bq.query AS blocking_query,
    bq.locktype,
    bq.relation,
    bq.state
FROM
    blocking_query bq;

-- Optional: Kill or cancel the blocking process
-- SELECT pg_cancel_backend(bq.blocking_pid);
-- OR
-- SELECT pg_terminate_session('session_id', 'true');

This SQL script first identifies processes that are holding locks which prevent a migration from proceeding. It then selects the exact query causing the blockage and its state, providing actionable insights to decide whether to wait, cancel, or kill the blocking process.

