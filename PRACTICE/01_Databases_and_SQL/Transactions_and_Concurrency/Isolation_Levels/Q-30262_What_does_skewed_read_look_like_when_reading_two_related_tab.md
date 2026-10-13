---
type: "practice"
difficulty: "Intermediate"
---

# What does 'skewed read' look like when reading two related tables under READ COMMITTED, and how do you make a multi-table read atomic in effect?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Phantoms and Double Booking](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Phantoms_and_Double_Booking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

A "skewed read" occurs when one of the tables being read is predominantly accessed based on a skewed distribution, leading to a disproportionate number of reads from this table. Under READ COMMITTED isolation level, a skewed read can lead to non-repeatable reads and dirty reads. To achieve atomic multi-table reads, we need to use techniques like locking or transactional read operations.

#### Triaging the Question

The question is asking about the behavior of a database under the READ COMMITTED isolation level when reading from two related tables that have a skewed distribution. Specifically, it wants you to explain how this can lead to issues and then suggest ways to ensure atomicity in multi-table reads. The interviewer might be probing your understanding of transactional behavior, isolation levels, and concurrency control.

#### Assumptions
We are working with a database using SQL.
There is a primary key linking the two tables (let's call them TableA and TableB).
TableA has a skewed distribution based on some column (e.g., an ID or date).

#### First Principles Solution
Skewed Read Behavior

Under READ COMMITTED isolation, each read operation sees the database state at the time of its execution. If one table (TableA) is heavily skewed and frequently accessed by TableB, a skewed read can occur. This happens when:

Read 1: A transaction starts and reads from TableA multiple times.
Write 1: Another transaction modifies data in TableA.
Read 2: The original transaction continues, reading TableA again to ensure consistency.

The second read (Read 2) will see the state of TableA as it was at the time of Read 2, which may not reflect changes made between the two reads (e.g., due to Write 1). This can lead to non-repeatable reads and dirty reads.

Making Multi-Table Reads Atomic

To ensure atomicity in multi-table reads under READ COMMITTED isolation, you need to use locking mechanisms. Here are a few approaches:

Row-Level Locking: Use explicit locks on the rows that TableB is accessing based on data from TableA.
Read Committed Snapshot Isolation (RCIS): Some databases support RCIS as an alternative to READ COMMITTED, which provides snapshot isolation without locking.
Serializable Reads: Use a SERIALIZABLE transaction level to ensure all operations are atomic and consistent.

Let's break down these solutions step by step:

Row-Level Locking

To implement row-level locks:

Start a transaction: BEGIN TRANSACTION;
Read from TableA while acquiring locks on the rows that TableB will access. For example, if you know which IDs in TableA correspond to entries in TableB, lock those specific rows using a command like: SELECT * FROM TableA WHERE ID IN (1, 2, 3) FOR UPDATE;
Proceed with reading from TableB: SELECT * FROM TableB WHERE A_ID = 1;
Read Committed Snapshot Isolation

This isolation level allows readers to see a consistent snapshot of the database state at the time the query started, without holding locks. However, it is not available in all databases and may have performance implications.

Serializable Reads

To use SERIALIZABLE transactions:

Start the transaction with SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
Perform your read operations within this transaction.
Ensure no other transaction modifies data that you are reading until the transaction commits or rolls back.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, skewed reads under READ COMMITTED can lead to non-repeatable and dirty reads. To ensure atomic multi-table reads, use row-level locking, consider RCIS if available, or switch to SERIALIZABLE isolation level. Each approach has trade-offs in terms of performance and complexity.

#### Common mistakes
include:

Not understanding the implications of READ COMMITTED.
Failing to realize that explicit locks are necessary for atomicity.
Overlooking the availability and impact of snapshot isolation levels like RCIS.

#### Follow-ups
might explore scenarios where these techniques fail or discussing alternative strategies, such as denormalization or indexing optimizations.

**Comparison:**

| ID | Value |
|----|-------|
| 1  | A     |
| 2  | B     |
| 3  | C     |
| A_ID | B_Value |
|------|---------|
| 1    | X       |
| 2    | Y       |
| 3    | Z       |

**Code:**

```sql
-- Example of row-level locking
BEGIN TRANSACTION;
SELECT * FROM TableA WHERE ID IN (1, 2, 3) FOR UPDATE; -- Lock specific rows in TableA
SELECT * FROM TableB WHERE A_ID = 1; -- Read from TableB while holding locks on TableA
COMMIT;
```

**Example input / output:**

```text
Input:
BEGIN TRANSACTION;
SELECT * FROM TableA WHERE ID IN (1, 2, 3) FOR UPDATE; -- Locks rows with IDs 1 and 2 in TableA
SELECT * FROM TableB WHERE A_ID = 1; -- Reads row with A_ID = 1 from TableB

Output:
-- Output of SELECT statements
TableA: [(ID=1, Value=A), (ID=2, Value=B)]
TableB: [A_ID=1, B_Value=X]
```

**Data:**

```json
{
  "TableA": [
    { "ID": 1, "Value": "A" },
    { "ID": 2, "Value": "B" },
    { "ID": 3, "Value": "C" }
  ],
  "TableB": [
    { "A_ID": 1, "B_Value": "X" },
    { "A_ID": 2, "B_Value": "Y" },
    { "A_ID": 3, "B_Value": "Z" }
  ]
}
```

**Flow:**

1. Start Transaction
2. Read from TableA with FORUPDATE
3. Write to TableA (e.g., Update)
4. Read from TableB
5. Commit or Rollback

