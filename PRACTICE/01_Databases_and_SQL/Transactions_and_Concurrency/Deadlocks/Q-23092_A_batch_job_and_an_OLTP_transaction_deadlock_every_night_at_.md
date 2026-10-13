---
type: "practice"
difficulty: "Deep"
---

# A batch job and an OLTP transaction deadlock every night at the same time; how would you diagnose the root cause and redesign to eliminate it?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Deadlock Causes and Diagnosis](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Deadlocks/Deadlock_Causes_and_Diagnosis.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To diagnose and resolve a deadlock issue between a batch job and an OLTP transaction, we first need to understand that deadlocks occur when two or more processes are blocked forever, waiting for each other's resources. In this scenario, the batch job and the OLTP transaction are likely competing for shared resources such as database locks, which can lead to a deadlock.

The core issue here is identifying why these transactions are accessing the same resources simultaneously without proper coordination. Let’s break down the steps required to diagnose and resolve the problem:

Identify Deadlocks: First, we need to confirm that deadlocks are indeed occurring by checking the database logs or using tools like SQL Server's sys.dm_exec_requests or Oracle's V$SESSION_WAIT. We will look for sessions where both the batch job and OLTP transaction are waiting on each other.
Analyze Locking Patterns: Once confirmed, we need to analyze the locking patterns of these transactions. This involves examining the SQL statements executed by the batch job and the OLTP transaction to understand how they acquire locks and in what order.
Determine Resource Contention: Identify which specific resources (tables, indexes, etc.) are causing the contention between the two transactions. We can use database performance monitoring tools or SQL queries to gather this information.
Design a Solution: Based on the analysis, we need to redesign the batch job and/or OLTP transaction logic to eliminate deadlock conditions. This could involve altering the order of resource acquisition, using different isolation levels, or implementing a retry mechanism.

Let’s go through each step in detail:

Identify Deadlocks

Use SQL Server's sys.dm_exec_requests:
SQL
SELECT
    r.session_id,
    t.text AS sql_text,
    r.status,
    r.wait_type,
    r.resource_description
FROM
    sys.dm_exec_requests r
CROSS APPLY
    sys.dm_exec_sql_text(r.sql_handle) t
WHERE
    r.status = 'suspended';
In Oracle, use V$SESSION_WAIT:
SQL
SELECT
    s.sid,
    s.serial#,
    w.event,
    w.p1text,
    w.p1,
    w.p2text,
    w.p2,
    w.p3text,
    w.p3,
    w.time_waited
FROM
    v$session_wait w,
    v$session s
WHERE
    w.session_id = s.sid AND
    event LIKE 'enq:%';

Analyze Locking Patterns

SQL Server:
SQL
SELECT
    t.name AS table_name,
    i.name AS index_name,
    l.resource_type,
    l.request_mode,
    r.session_id,
    r.status
FROM
    sys.dm_tran_locks l
JOIN
    sys.tables t ON l.resource_associated_entity_id = t.object_id
JOIN
    sys.indexes i ON l.resource_associated_entity_id = i.object_id AND l.index_page = i.index_id
JOIN
    sys.dm_exec_requests r ON l.request_session_id = r.session_id;
Oracle:
SQL
SELECT
    s.sid,
    s.serial#,
    t.table_name,
    t.index_name,
    l.locked_mode
FROM
    v$session s,
    dba_data_locks d,
    dba_tab_columns c,
    all_tables t,
    dba_ind_columns i
WHERE
    s.sid = d.session_id AND
    s.sql_address = c.owner || '.' || c.table_name AND
    d.object_id = t.table_id AND
    d.index_id = i.column_id;

Determine Resource Contention

We need to identify the specific tables and indexes involved in the deadlock. This can be done by analyzing the SQL statements that are causing the contention.

Design a Solution

Modify the batch job to ensure it acquires locks on resources in a consistent order.
Use Oracle's FOR UPDATE clause or SQL Server’s explicit transaction management to control locking behavior.
Implement retry logic with exponential backoff for transactions that might be prone to deadlocks.

By following these steps, we can diagnose the root cause of the deadlock and design a solution that prevents it from occurring in the future. Common mistakes include not thoroughly analyzing the locking patterns or making changes without fully understanding the impact on system performance.

**Code:**

```sql
-- SQL Server Example
SELECT
    r.session_id,
    t.text AS sql_text,
    r.status,
    r.wait_type,
    r.resource_description
FROM
    sys.dm_exec_requests r
CROSS APPLY
    sys.dm_exec_sql_text(r.sql_handle) t
WHERE
    r.status = 'suspended';

-- Oracle Example
SELECT
    s.sid,
    s.serial#,
    t.table_name,
    t.index_name,
    l.locked_mode
FROM
    v$session s,
    dba_data_locks d,
    dba_tab_columns c,
    all_tables t,
    dba_ind_columns i
WHERE
    s.sid = d.session_id AND
    s.sql_address = c.owner || '.' || c.table_name AND
    d.object_id = t.table_id AND
    d.index_id = i.column_id;
```

**Flow:**

1. Identify Deadlocks
2. Analyze Locking Patterns
3. Determine ResourceContention
4. Design a Solution

