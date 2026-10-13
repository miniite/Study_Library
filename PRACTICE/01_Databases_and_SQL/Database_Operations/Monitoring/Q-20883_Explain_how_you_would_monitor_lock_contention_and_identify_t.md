---
type: "practice"
difficulty: "Deep"
---

# Explain how you would monitor lock contention and identify the blocking session in a live incident.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Database Health Signals](<../../../../01_Databases_and_SQL/Database_Operations/Monitoring/Database_Health_Signals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively monitor lock contention and identify the blocking session during an incident, we need to leverage database performance monitoring tools and understand the underlying SQL statements involved. The goal is to quickly pinpoint the source of contention to minimize downtime and ensure smooth operations.

#### Triage

The question asks for a method to monitor and resolve lock contention issues in real-time. The key points being probed are:

Understanding Lock Contention: What types of locks exist, their common causes, and how they affect performance.
Identifying Blocking Sessions: How to trace the sequence of events leading up to a deadlock or significant delay.
Real-Time Monitoring: Tools and techniques for proactive monitoring versus reactive incident handling.

#### First-Principles Solution

Understanding Lock Types:

Shared (S) Locks: Allow concurrent read operations but prevent writes.
Exclusive (X) Locks: Permit exclusive access, preventing any other sessions from accessing the resource in either a read or write mode.
Update (U) Locks: A variant of shared locks that are used to allow concurrent reads while preparing for an eventual exclusive lock.

Monitoring Tools and Metrics:

Performance Schema: MySQL has a built-in Performance Schema that can be enabled to monitor lock events in real-time.
DBMS-specific Tracing Tools: PostgreSQL, for example, provides pg_locks table which can be queried directly or via tools like psql.
Third-Party Monitoring Solutions: Tools like New Relic, Datadog, and Prometheus offer comprehensive monitoring capabilities.

Identifying Contention:

Lock Wait Statistics: Track the number of times locks are waited for and the duration.
Session Traces: Use tracing tools to capture detailed information about blocking sessions.

Investigating Blocking Sessions:

INFORMATION_SCHEMA.PROCESSLIST: For MySQL, this table can be used to identify active sessions and their current state.
Lock Analysis Scripts: Write custom scripts that analyze pg_locks for PostgreSQL or equivalent tables in other DBMS.

#### Example
Workflow

Enable Performance Schema:

SQL
SET GLOBAL performance_schema=ON;

Query Lock Wait Statistics:

SQL
SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS WHERE blocking_session IS NOT NULL LIMIT 50;

Analyze Session Details:

SQL
SELECT * FROM INFORMATION_SCHEMA.PROCESSLIST WHERE ID = <blocking_session_id>;

Custom Lock Analysis Script:

SQL
CREATE PROCEDURE FindBlockingSessions()
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE session_id INT;
    DECLARE blocking_locks CURSOR FOR
        SELECT blocking_session FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS;
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;

    OPEN blocking_locks;
    read_loop: LOOP
        FETCH blocking_locks INTO session_id;
        IF done THEN LEAVE read_loop; END IF;
        -- Further analysis on `session_id`
    END LOOP;
    CLOSE blocking_locks;
END;

#### Closing Summary
and Common Mistakes/Follow-Ups

By leveraging the Performance Schema, custom scripts, and tracing tools, we can effectively monitor lock contention in a live database. Common mistakes include:

Ignoring fine-grained locking mechanisms.
Failing to proactively set up monitoring solutions before an incident occurs.

#### Follow-ups
might involve discussing how to implement and maintain these solutions in production environments or how to integrate them with CI/CD pipelines for automated testing.

**Comparison:**

| Blocking Session Details |
|--------------------------|
| `blocking_session_id`    |
| `requested_lock_type`    |
| `blocking_transaction_id`|
| `lock_mode`              |

**Code:**

```sql
-- Enable Performance Schema
SET GLOBAL performance_schema=ON;

-- Query Lock Wait Statistics
SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS WHERE blocking_session IS NOT NULL LIMIT 50;

-- Find Blocking Sessions Procedure
CREATE PROCEDURE FindBlockingSessions()
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE session_id INT;
    DECLARE blocking_locks CURSOR FOR
        SELECT blocking_session FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS;
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;

    OPEN blocking_locks;
    read_loop: LOOP
        FETCH blocking_locks INTO session_id;
        IF done THEN LEAVE read_loop; END IF;
        -- Further analysis on `session_id`
    END LOOP;
    CLOSE blocking_locks;
END;
```

**Example input / output:**

```text
Input:
SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS WHERE blocking_session IS NOT NULL LIMIT 50;

Output:
+-----------------+-----------------------+------------------------+----------+
| blocking_session| requested_lock_type   | blocking_transaction   | lock_mode|
+-----------------+-----------------------+------------------------+----------+
| 123456789       | S                     | 987654321              | X        |
+-----------------+-----------------------+------------------------+----------+

Input:
CALL FindBlockingSessions();

Output:
-- Execution of the stored procedure, identifies blocking sessions and their details
```

**Data:**

```plaintext
-- Example data from INFORMATION_SCHEMA.INNODB_LOCK_WAITS
blocking_session | requested_lock_type  | blocking_transaction  | lock_mode
-----------------|---------------------|----------------------|----------
123456789        | S                   | 987654321             | X
```

**Flow:**

1. Yes
2. No
3. Enable Performance Schema
4. Is Lock Contention Detected?
5. Analyze Lock Waits andBlocking Sessions
6. Investigate Session Details
7. Implement Custom Scripts forAnalysis
8. Monitor Regularly and Set UpAlerts

