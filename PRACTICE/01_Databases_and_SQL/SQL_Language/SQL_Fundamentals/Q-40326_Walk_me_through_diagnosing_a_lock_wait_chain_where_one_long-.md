---
type: "practice"
difficulty: "Expert"
---

# Walk me through diagnosing a lock wait chain where one long-running analytical query blocks hundreds of short OLTP writes. What views reveal the blocking tree and how do you break it safely?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ SKIP LOCKED Job Queues](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/SKIP_LOCKED_Job_Queues.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To diagnose and resolve a lock wait chain in a database, particularly when an analytical query is blocking hundreds of short transactional writes, we need to identify the root cause and explore safe methods to mitigate the issue. This involves understanding the locking behavior, analyzing the queries involved, and potentially refactoring or optimizing the database schema.

Triaging the Problem
What is Being Probed:

The interviewer is likely probing your ability to understand concurrency issues in databases, your skill in diagnosing complex blocking scenarios, and your knowledge of SQL and transactional concepts. You will need to demonstrate a clear understanding of lock waits, isolation levels, query execution plans, and potential optimizations.

#### Assumptions
The database uses SQL.
There are two types of queries: long-running analytical queries and short OLTP writes.
The system is experiencing performance issues due to blocking.

#### First-Principles Solution
Identify the Blocking: Use SHOW ENGINE INNODB STATUS or equivalent commands (like pg_locks in PostgreSQL) to find out which transaction is causing the lock wait chain.
Analyze the Query Plans: Examine the query execution plans of both the analytical and OLTP queries using EXPLAIN statements.
Refactor Queries: Optimize or modify the queries to reduce locking impact.
Adjust Transaction Isolation Levels: Consider changing the isolation level if it is causing excessive locking.
Partitioning and Indexing: Improve schema design by partitioning tables and adding appropriate indexes.
Step-by-Step Solution

Identify the Blocking:

Run SHOW ENGINE INNODB STATUS; in MySQL to get detailed information about locks.
Look for the "TRANSACTIONS" section which will show you the current transactions, their states, and the locks they hold or wait on.

Analyze the Query Plans:

Use EXPLAIN on both types of queries to understand how they are executed and identify potential bottlenecks.
For example, in MySQL:
SQL
EXPLAIN SELECT * FROM sales WHERE date > '2023-01-01';
Check if the query uses full table scans or can be optimized with indexes.

Refactor Queries:

Simplify complex queries to reduce their execution time and locking duration.

#### Cons
ider breaking down large analytical queries into smaller chunks that run concurrently.
Use partitioning in tables where data is accessed based on certain criteria, e.g., by date ranges.

Adjust Transaction Isolation Levels:

Change the isolation level from REPEATABLE READ to READ COMMITTED if applicable, which can reduce locking contention.
Be cautious as this might introduce inconsistencies in read-only transactions.

Partitioning and Indexing:

Partition tables by date or other relevant criteria to distribute data more evenly.
Add indexes on frequently queried columns to speed up write operations.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By systematically identifying the blocking transactions, analyzing query execution plans, refactoring queries, adjusting transaction isolation levels, and optimizing schema design, we can effectively mitigate lock wait chains. Common mistakes include not fully understanding the locking behavior of the database, making changes without thorough testing, and ignoring the trade-offs between different optimization strategies.

#### Follow-Ups
How do you ensure that your optimizations do not affect the correctness of the data?
What performance metrics would you use to validate that the changes have been effective?
Can you provide an example of a scenario where partitioning could lead to better performance in this context?

**Comparison:**

```
tableSchema sales {
  id INT,
  date DATE,
  amount DECIMAL(10,2)
}
```

**Code:**

```sql
-- Example SHOW ENGINE INNODB STATUS output
SHOW ENGINE INNODB STATUS;

-- Example EXPLAIN output for query analysis
EXPLAIN SELECT * FROM sales WHERE date > '2023-01-01';
```

**Example input / output:**

```text
-- Input: SHOW ENGINE INNODB STATUS;
-- Output:
# Transaction fetches:

+--------+----------------------+------------+--------------------------+-----------------------------+
| ID     | Mysql_thread_id      | State      | Lock                     | Waiter                       |
+--------+----------------------+------------+--------------------------+-----------------------------+
| 123456 | 7890                | LOCK WAIT  | record in set            | query                        |
| 123457 | 7891                | LOCK WAIT  | record in set            | query                        |
| 123458 | 7892                | LOCK WAIT  | record in set            | query                        |
+--------+----------------------+------------+--------------------------+-----------------------------+

-- Input: EXPLAIN SELECT * FROM sales WHERE date > '2023-01-01';
-- Output:
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
| 1  | SIMPLE      | sales | NULL       | index | PRIMARY       | date    | 5       | NULL | 2347 | 100.00   | Using where |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
```

**Data:**

```json
{
  "sales": {
    "id": [1, 2, 3],
    "date": ["2023-01-01", "2023-01-01", "2023-01-05"],
    "amount": [100, 200, 300]
  }
}
```

**Flow:**

1. Yes
2. Yes
3. Yes
4. No
5. Identify Blocking Transactions
6. Use SHOW ENGINE INNODBSTATUS?
7. Analyze Query Plans
8. Refactor Queries?
9. Adjust Transaction IsolationLevels
10. Partition and Index Tables?
11. Validate Changes
12. End

