---
type: "practice"
difficulty: "Deep"
---

# What is a periodic snapshot fact table and when is it useful, for example daily account balances?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Transaction vs Snapshot Facts](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Transaction_vs_Snapshot_Facts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A periodic snapshot fact table in data modeling refers to a type of fact table that captures historical states of an entity at specific points in time. This is particularly useful when you need to track changes over time, such as daily account balances. Let's break down the concept and explore its utility through a detailed first-principles solution.

#### Triage

The question asks about periodic snapshot fact tables and their usefulness, specifically with an example of tracking daily account balances. The core idea is understanding how to model temporal data accurately so that historical states can be queried efficiently. The interviewer might be probing your knowledge of data modeling best practices, especially in scenarios where you need to store and retrieve historical information.

Key Assumptions
We have a dimension table representing accounts.
Accounts can change over time (e.g., balances, account details).
Historical snapshots are needed for auditing or reporting purposes.

#### First-Principles Solution

Identify the Fact Table Structure:

The periodic snapshot fact table will store historical records of account balances at specific points in time.
Each row represents an account balance at a particular date.

Define Dimensions and Facts:

Dimensions: AccountID, Date.
Facts: Balance.

Create the Schema:

SQL
CREATE TABLE AccountSnapshots (
    AccountID INT,
    SnapshotDate DATE PRIMARY KEY,
    Balance DECIMAL(10, 2)
);

Data Insertion and Update Mechanism:

Each day, a new row is inserted with the updated balance for each account.
Alternatively, balances can be updated using an upsert mechanism (INSERT or REPLACE).

Querying Historical Data:

To retrieve historical data, you can use standard SQL queries:
SQL
SELECT AccountID, SnapshotDate, Balance FROM AccountSnapshots WHERE AccountID = 123 AND SnapshotDate BETWEEN '2023-01-01' AND '2023-06-30';

#### Closing Summary
and Common Mistakes/Follow-Ups

Periodic snapshot fact tables are useful for scenarios where historical data is critical, such as financial reporting or compliance. They ensure that you can query any point in time accurately by storing snapshots at regular intervals.

#### Common Mistakes

Forgetting to handle date ranges properly.
Not ensuring uniqueness on the SnapshotDate column.
Overwriting old records instead of inserting new ones, leading to data loss.

#### Follow-Ups

Discussing potential performance optimizations (e.g., partitioning by date).

#### Cons
idering whether a star or snowflake schema would be more appropriate in this context.
Exploring how to handle large volumes of historical data efficiently.

**Code:**

```sql
CREATE TABLE AccountSnapshots (
    AccountID INT,
    SnapshotDate DATE PRIMARY KEY,
    Balance DECIMAL(10, 2)
);
```

