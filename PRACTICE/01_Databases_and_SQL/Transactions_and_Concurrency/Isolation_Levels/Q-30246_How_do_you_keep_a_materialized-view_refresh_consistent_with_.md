---
type: "practice"
difficulty: "Deep"
---

# How do you keep a materialized-view refresh consistent with concurrent base-table writes? Which isolation level do you run the refresh under?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Isolation with Read Replicas](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/Isolation_with_Read_Replicas.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a materialized view (MV) refresh remains consistent with concurrent base table writes, we need to understand how different isolation levels affect transaction processing. In a database environment where multiple transactions are writing to the base tables while an MV refresh is running, maintaining consistency can be challenging. We will explore this problem by first understanding what each isolation level guarantees and then determining which one ensures that the MV refresh sees a consistent snapshot of the data.

#### Triage
What's Being Probed

The question is asking about how to keep a materialized view refreshed in a way that it accurately reflects the current state of the base tables, despite concurrent writes. The interviewer is probing your understanding of isolation levels and their implications on transaction processing and MV consistency.

Key Assumptions
We are using a relational database management system (RDBMS) like PostgreSQL or SQL Server.
The materialized view contains summarized data derived from the base tables.
Concurrent writes to the base tables could potentially alter the state of the data that the materialized view depends on.

#### First-Principles Solution
Understanding Isolation Levels

Isolation levels determine how transactions see each other's changes. In a database, four common isolation levels are used: Read Uncommitted (RU), Read Committed (RC), Repeatable Read (RR), and Serializable (Serializable).

Read Uncommitted: This level allows dirty reads; a transaction can read data written by uncommitted transactions.
Read Committed: A transaction can only see the committed changes of other transactions. It prevents dirty reads but allows non-repeatable reads.
Repeatable Read: Ensures that once a value is read, it will not change until the current transaction ends. It prevents non-repeatable reads and phantom reads.
Serializable: Provides full isolation; transactions run as if they are the only ones executing in the database.
Determining the Appropriate Isolation Level

For ensuring consistency during an MV refresh, we need to guarantee that the refresh process sees a consistent snapshot of the base tables. The appropriate isolation level for this scenario is Repeatable Read (RR), because:

It ensures that once a value is read, it will not change until the current transaction ends.
This means if the MV refresh starts reading the base table data, no concurrent writes can alter those values during the refresh process.
Step-by-Step Solution

Start the Materialized View Refresh with an Isolation Level of RR:

SQL
BEGIN ISOLATION LEVEL REPEATABLE READ;
-- Start the MV refresh logic here

Read the Base Table Data: During the refresh, read all necessary data from the base tables to compute the new state of the materialized view.

Update the Materialized View: Once the computation is complete and all reads are in a consistent snapshot, update the materialized view with the computed data.

Commit the Changes: After updating the materialized view, commit the transaction to finalize the changes.

#### Example Scenario

Suppose we have a base table Sales that logs sales transactions, and a materialized view MonthlySummary that summarizes monthly sales by region. If concurrent writes are happening in Sales, we want to ensure that the summary accurately reflects those changes during the refresh.

SQL
BEGIN ISOLATION LEVEL REPEATABLE READ;
-- Read all necessary data from Sales table
SELECT * FROM Sales WHERE sale_date BETWEEN '2023-01-01' AND '2023-01-31';

-- Compute the summary based on the read data

-- Update the MonthlySummary materialized view with the computed data
UPDATE MonthlySummary SET total_sales = ...;

COMMIT;

#### Closing Summary
and Common Mistakes/Follow-Ups

Using Repeatable Read ensures that the MV refresh sees a consistent snapshot of the base tables, preventing issues like non-repeatable reads or dirty reads. However, it's important to note that this isolation level might cause performance overhead due to the snapshot locking mechanism.

#### Common mistakes
include:

Using Read Committed which can lead to inconsistent snapshots.
Not starting the refresh transaction with a consistent isolation level at all.
Failing to account for concurrency issues and ensuring that all reads are in a repeatable state.

In follow-up questions, the interviewer might ask about trade-offs between different isolation levels or how other MV consistency techniques (like snapshot isolation) can be applied in this scenario.

**Comparison:**

| Region | Total Sales |
|--------|-------------|
| North  | 1000        |
| South  | 1500        |

**Code:**

```sql
BEGIN ISOLATION LEVEL REPEATABLE READ;
-- Read data from base table
SELECT * FROM Sales WHERE sale_date BETWEEN '2023-01-01' AND '2023-01-31';

-- Compute the summary based on read data

-- Update materialized view with computed data
UPDATE MonthlySummary SET total_sales = ...;

COMMIT;
```

**Example input / output:**

```text
-- Before refresh
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT * FROM Sales WHERE sale_date BETWEEN '2023-01-01' AND '2023-01-31';

-- Compute the summary based on read data (this step would involve complex logic)
UPDATE MonthlySummary SET total_sales = ...;

COMMIT;
```

**Data:**

```json
{
  "Sales": [
    {"sale_id": 1, "region": "North", "amount": 1000, "sale_date": "2023-01-15"},
    {"sale_id": 2, "region": "South", "amount": 1500, "sale_date": "2023-01-20"}
  ],
  "MonthlySummary": [
    {"region": "North", "total_sales": 1000},
    {"region": "South", "total_sales": 1500}
  ]
}
```

**Flow:**

1. Start Refresh Transaction
2. Isolation Level?
3. Repeatable Read
4. Read Base Table Data
5. Compute Summary Based onReads
6. Update Materialized View
7. Commit Changes
8. End Refresh Transaction

