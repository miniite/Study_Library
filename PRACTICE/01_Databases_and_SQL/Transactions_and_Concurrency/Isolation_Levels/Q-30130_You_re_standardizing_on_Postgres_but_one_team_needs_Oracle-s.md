---
type: "practice"
difficulty: "Expert"
---

# You're standardizing on Postgres but one team needs Oracle-style SERIALIZABLE-as-snapshot semantics for a ported app. How do you bridge the gap without rewriting their transactions?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Serializable Performance](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/Serializable_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of providing Oracle-style SERIALIZABLE-as-snapshot semantics in Postgres, we need to understand the nuances between isolation levels and how they manifest differently in these two database systems. The key is to leverage PostgreSQL's advanced features such as read committed snapshots and carefully crafted transactional patterns to mimic the behavior without full rewriting.

Firstly, let's define the requirements and what SERIALIZABLE-as-snapshot means. In Oracle, SERIALIZABLE isolation ensures that transactions are isolated from uncommitted changes by other transactions, mimicking a serial execution order. This is different from Postgres’ default read committed level, which can read dirty data (uncommitted changes).

To bridge this gap in PostgreSQL, we need to:

Understand the Oracle Semantics: Oracle's SERIALIZABLE-as-snapshot isolates transactions by ensuring that each transaction sees a consistent snapshot of the database state at the start of its execution.
Postgres Default Behavior: Postgres’ default read committed level allows dirty reads (reading uncommitted data). We need to control this behavior.

Here’s how we can achieve this:

Step-by-Step Solution

Use Read Committed Snapshot Isolation:

PostgreSQL supports a feature called "Read Committed Snapshot" which ensures that transactions do not see uncommitted changes from other concurrent transactions. This is a key step towards replicating Oracle's SERIALIZABLE-as-snapshot semantics.
SQL
SET TRANSACTION ISOLATION LEVEL READ COMMITTED SNAPSHOT;

Isolation Level Control:

We can set the isolation level to READ COMMITTED with snapshot support, but for critical transactions that require full SERIALIZABLE behavior, we might need additional logic.
SQL
SET TRANSACTION ISOLATION LEVEL READ COMMITTED SNAPSHOT;

Transaction Management:

For critical sections of the transaction where full isolation is needed, we can use BEGIN to start a new transaction and explicitly manage locks to ensure no dirty data is read.

#### Example
SQL Implementation

Here’s an example scenario where you might need more control over transaction management:

SQL
-- Start a new transaction with explicit control
BEGIN;

-- Use snapshot isolation
SET TRANSACTION ISOLATION LEVEL READ COMMITTED SNAPSHOT;

-- Perform critical operations that require full SERIALIZABLE behavior
SELECT * FROM some_table WHERE condition FOR UPDATE;

-- Ensure data consistency by locking rows
UPDATE some_table SET column = value WHERE id = 1 AND some_condition;

-- Commit or rollback the transaction as needed
COMMIT;

#### Common Mistakes
and Follow-Ups
Not Understanding Isolation Levels: Misunderstanding how different isolation levels work can lead to incorrect assumptions about data consistency.
Overuse of SERIALIZABLE Level: Overusing the SERIALIZABLE level can cause performance issues due to excessive locking.
Failure to Use FOR UPDATE: Missing critical sections where FOR UPDATE is necessary can lead to inconsistent reads.

#### Closing Summary

In conclusion, by leveraging PostgreSQL’s advanced features like read committed snapshots and carefully managing transaction isolation levels, we can bridge the gap between Oracle's SERIALIZABLE-as-snapshot semantics and Postgres. This approach ensures that transactions see a consistent snapshot of the database state while avoiding unnecessary full SERIALIZABLE level usage.

**Code:**

```sql
BEGIN;
SET TRANSACTION ISOLATION LEVEL READ COMMITTED SNAPSHOT;

SELECT * FROM some_table WHERE condition FOR UPDATE;
UPDATE some_table SET column = value WHERE id = 1 AND some_condition;

COMMIT;
```

