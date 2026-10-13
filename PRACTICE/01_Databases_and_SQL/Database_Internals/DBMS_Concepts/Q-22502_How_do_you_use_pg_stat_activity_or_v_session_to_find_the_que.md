---
type: "practice"
difficulty: "Deep"
---

# How do you use pg_stat_activity or v$session to find the query and lock currently blocking a stuck transaction?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Lock Granularity](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Lock_Granularity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To diagnose and resolve stuck transactions in PostgreSQL, you can use pg_stat_activity (or Oracle's equivalent v$session) to identify sessions that are holding locks or waiting on them. This involves understanding how session states and lock information are represented, identifying the problematic session(s), and deciding on appropriate actions.

#### Triage
The question probes your knowledge of PostgreSQL's internal state tracking mechanisms and your ability to interpret system views for performance troubleshooting. You need to assume a stuck transaction is causing some form of blocking or is itself being blocked by another session. The core idea is to use pg_stat_activity (or v$session) to gather detailed information about current sessions, their states, query execution status, and lock information.

#### First-principles solution
Start with identifying the problematic session using pg_stat_activity. This view shows active backend processes in the PostgreSQL database. You can filter by state to find sessions that are stuck or waiting (states like 'active', 'idle in transaction', or 'fastpath lock wait').

SQL
SELECT
    pid,
    usename,
    query_start,
    query,
    state_detail
FROM
    pg_stat_activity
WHERE
    state = 'active' AND
    state_detail LIKE '%lock%';

Further drilling down, use pg_locks to see the exact locks held by each session:

SQL
SELECT
    l.pid AS blocking_pid,
    a.query as blocking_query,
    l.relation::regclass AS blocked_table,
    p.usename AS blocking_user,
    q.query_start AS query_start_time,
    q.state_detail AS state_detail
FROM
    pg_locks l
JOIN
    pg_stat_activity p ON p.pid = l.pid
JOIN
    (SELECT * FROM pg_stat_activity WHERE state = 'active' AND state_detail LIKE '%lock%') q ON q.pid = l.relation::regclass;

This query joins pg_locks with pg_stat_activity to get more context on the blocking transaction. Finally, if you identify a session that is stuck or blocking others, take appropriate action (e.g., kill the session, optimize queries).

#### Closing summary
Use pg_stat_activity and pg_locks to diagnose stuck transactions and their impacts. Always consider the broader implications of killing sessions and opt for non-intrusive actions first.

#### Common mistakes/follow-ups

Not considering all states in pg_stat_activity.
Not using pg_locks to get detailed lock information.
Not understanding the full impact of interrupting a session.
Failing to follow up with query optimization or database tuning after resolving blocking issues.

**Code:**

```

SELECT pid, usename, query_start, query, state_detail FROM pg_stat_activity WHERE state = 'active' AND state_detail LIKE '%lock%';

CODE
Copy

```sql
SELECT
    l.pid AS blocking_pid,
    a.query as blocking_query,
    l.relation::regclass AS blocked_table,
    p.usename AS blocking_user,
    q.query_start AS query_start_time,
    q.state_detail AS state_detail
FROM
    pg_locks l
JOIN
    pg_stat_activity p ON p.pid = l.pid
JOIN
    (SELECT * FROM pg_stat_activity WHERE state = 'active' AND state_detail LIKE '%lock%') q ON q.pid = l.relation::regclass;
```

