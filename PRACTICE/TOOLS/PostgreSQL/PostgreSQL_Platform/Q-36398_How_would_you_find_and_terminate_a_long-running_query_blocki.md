---
type: "practice"
difficulty: "Deep"
---

# How would you find and terminate a long-running query blocking others?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Finding Expensive Queries](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Finding_Expensive_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To find and terminate a long-running query in PostgreSQL, we need to diagnose the issue, ensure minimal impact on other queries, and then take appropriate action. This process involves understanding the query's nature, identifying the problematic query, and stopping it gracefully while minimizing disruption.

#### Triage

The question is essentially asking how to manage and resolve performance issues caused by a long-running or stuck query that might block others. The interviewer is probing your knowledge of PostgreSQL internals, your ability to diagnose resource bottlenecks, and your understanding of concurrency control mechanisms in databases. Key assumptions include:

You have access to the database server.
You can run administrative commands on the server.
You are familiar with common tools like psql or pgAdmin.

#### First Principles Solution

Identify the Query:

Use the pg_stat_activity view to find running queries that might be blocking others.
SQL
SELECT * FROM pg_stat_activity WHERE query LIKE '%suspicious pattern%';
Alternatively, you can use psql with the \qset terminate command enabled:
SH
psql -c "\qset terminate"
\l+  # List databases and look for long-running queries

Analyze Resource Usage:

Check CPU and memory usage of the query using pg_stat_activity or top/htop.
SQL
SELECT * FROM pg_stat_activity WHERE state = 'active' ORDER BY query_start;
Look for queries that have been running for an unusually long time.

Determine Blocking:

Use pg_locks to identify if the long-running query is blocking other transactions.
SQL
SELECT * FROM pg_locks WHERE relation = (SELECT oid FROM pg_stat_activity WHERE pid = <pid>) AND granted;
If a lock is preventing other queries from proceeding, it indicates a potential issue.

Terminate the Query:

Use pg_cancel_backend or pg_terminate_backend to stop the query.
SQL
SELECT pg_cancel_backend(<pid>);
-- Or for immediate termination:
SELECT pg_terminate_backend(<pid>);
Replace <pid> with the process ID of the long-running query.

Monitor and Clean Up:

Ensure that the terminated query does not leave behind any locks or transactions in a problematic state.
SQL
SELECT * FROM pg_locks WHERE relation = (SELECT oid FROM pg_stat_activity WHERE pid = <pid>);
If necessary, run VACUUM to clean up any affected tables.

#### Closing Summary
and Common Mistakes

#### Common Mistakes

Forgetting to check the pg_stat_activity view or using it incorrectly.
Terminating the wrong query due to misidentification.
Not monitoring for potential side effects of terminating a long-running transaction.

#### Follow-Ups

What tools would you use if you needed to monitor performance over time?
How do you ensure that similar issues don’t happen in the future?

**Code:**

```sql
-- Example SQL queries
SELECT * FROM pg_stat_activity WHERE state = 'active' ORDER BY query_start;
SELECT * FROM pg_locks WHERE relation = (SELECT oid FROM pg_stat_activity WHERE pid = <pid>) AND granted;

-- Terminate a backend process
SELECT pg_terminate_backend(<pid>);
```

**Example input / output:**

```text
psql -c "\qset terminate"
\l+  # List databases and look for long-running queries

SELECT * FROM pg_stat_activity WHERE state = 'active' ORDER BY query_start;

# Example output
| pid | usename  | application_name | state | query |
|-----|---------|-----------------|-------|-------|
| 1234| user    | -               | active| SELECT ... |

SELECT * FROM pg_locks WHERE relation = (SELECT oid FROM pg_stat_activity WHERE pid = 1234) AND granted;

# Example output
| relation_id | grantee_tuple | requested | granted |
|------------|---------------|-----------|---------|
| 10768       | (1234,10001)  | 'SELECT'  | t       |

pg_terminate_backend(1234);
```

**Flow:**

1. Yes
2. No
3. Identify Query
4. Is the query blocking others?
5. Terminate Query
6. Monitor Other Transactions
7. Maintain Database Cleanliness
8. Maintain Monitoring

