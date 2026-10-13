---
type: "practice"
difficulty: "Intermediate"
---

# Explain what happens to visibility if you SET TRANSACTION ISOLATION LEVEL after the first query has already run in PostgreSQL.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Setting Isolation Levels](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with database transactions in PostgreSQL, understanding how setting a transaction isolation level affects subsequent queries is crucial. In this scenario, we are dealing with the SET TRANSACTION ISOLATION LEVEL command and its impact on query visibility after an initial query has run.

#### Triage
of the Question

The question asks about the behavior of visibility in a transaction once you change the isolation level after the first query has already executed. The core concept here is understanding how different isolation levels (RU, RC, RR, and Serializable) affect read operations within transactions. We need to consider how these settings can influence the consistency and concurrency of data accessed by multiple concurrent transactions.

#### First-Principles Solution
Initial Context: Transaction Isolation Levels in PostgreSQL

PostgreSQL supports several isolation levels:

Read Uncommitted (RU) - Allows reading uncommitted changes.
Read Committed (RC) - Only allows reading committed changes.
Repeatable Read (RR) - Ensures that multiple reads within the same transaction see the same data, but does not prevent dirty or non-repeatable reads.
Serializable - Provides complete isolation by serializing transactions.
Changing Isolation Levels Mid-Transaction

When you set a new isolation level using SET TRANSACTION ISOLATION LEVEL while in an ongoing transaction:

The change takes effect from the point it is issued onward, not retroactively to previous queries.
Any locks held before changing the isolation level remain unaffected by the new setting.
Detailed Example: Scenario with Isolation Level Change

#### Cons
ider a scenario where you have two transactions ( T_1 ) and ( T_2 ), both reading from the same table orders.

Transaction ( T_1 ): Starts in Read Committed mode, runs a query to fetch all orders.
After fetching some results, it issues SET TRANSACTION ISOLATION LEVEL Serializable.
Meanwhile, transaction ( T_2 ) inserts a new order.

Here’s how the transactions behave under different isolation levels:

Before Changing Isolation Level

Transaction ( T_1 ):

Initially in Read Committed mode.
Reads data as it was when the query started (committed state).

Transaction ( T_2 ):

Inserts an order, which is committed and visible to other reads after its transaction commit.
After Changing Isolation Level

Transaction ( T_1 ):

Locks held before issuing SET TRANSACTION ISOLATION LEVEL remain as they are.
From now on, the transaction operates in Serializable mode. Any new reads or writes will be isolated from concurrent transactions until it commits.

#### Closing Summary
and Common Mistakes/Follow-Ups
Key Takeaway: Changing isolation levels mid-transaction affects only future queries, not those already running.
Common Mistake: Assuming the change applies retroactively to previous queries can lead to misunderstandings of transactional behavior.

#### Follow-Up
Questions: How does this affect deadlock detection? What about concurrent transactions and their visibility?

**Comparison:**

```
executed
held
Orders
Transactions
in
Locks
on
```

**Example input / output:**

| Initial Query | Change Isolation Level |
|---------------|-----------------------|
| `BEGIN;`       | `SELECT * FROM orders;` |
| `-- Fetches data as of the start time` |  |
| `SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;` | `INSERT INTO orders (id, amount) VALUES (101, 50.0);` - Would block or fail due to locking constraints. |

**Flow:**

1. Yes
2. Begin Transaction
3. Query 1 (Read Committed)
4. Isolation Level Changed?
5. Query 2 (Serializable Mode)
6. Commit or Rollback
7. Insert Operation (Blocked)

