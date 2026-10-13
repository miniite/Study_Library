---
type: "practice"
difficulty: "Expert"
---

# How does the choice of isolation level affect index behaviour under concurrent inserts — specifically, why do gap locks appear only for some index types?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Phantoms and Double Booking](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Phantoms_and_Double_Booking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: The choice of isolation level significantly influences how indexes behave under concurrent insert operations, particularly in terms of when and where gap locks are applied. Gap locks primarily appear due to non-unique B-tree index types (e.g., clustered or non-clustered), whereas unique indexes like primary keys do not require them.

#### Triage
The question is asking about the specific behavior of gap locks (a type of lock) in different isolation levels when dealing with concurrent insert operations. Specifically, it probes why some indexes use gap locks while others do not. This involves understanding how transaction isolation levels affect locking mechanisms and the nature of index types.

#### Assumptions

We are discussing a relational database system that supports various isolation levels.
The focus is on non-unique B-tree index types, as unique indexes (like primary keys) typically don't require gap locks.

#### First Principles Solution

Understanding Isolation Levels and Locks

Isolation levels define the level of independence between transactions in a database. The main isolation levels are:

Read Uncommitted (RU): Dirty reads allowed.
Read Committed (RC): Non-repeatable reads and phantom reads possible.
Repeatable Read (RR): Repeatable reads but not repeatable updates.
Serializable (S): Ensures serializability, no concurrency issues.

Gap locks are a type of row-level lock used to prevent non-unique indexes from being split during concurrent insert operations. They are applied on the gap between rows in an index structure.

Impact of Index Types
1. Non-Unique B-Tree Indexes (Clustered and Non-Clustered)

In databases like SQL Server, MySQL, or PostgreSQL, non-unique B-tree indexes can lead to the application of gap locks during concurrent insert operations. This is because these indexes do not ensure uniqueness across all rows in a table.

Clustering: In clustered indexing, each row is stored at an exact location based on its index key. If two transactions attempt to insert rows with overlapping keys, a gap lock might be necessary to maintain the integrity of the index structure.

Non-Clustered Indexes: Even though non-clustered indexes point to the actual data in the table, they still need to ensure consistent behavior during concurrent operations.

2. Unique B-Tree Indexes (Primary Keys)

Unique B-tree indexes, such as primary keys, do not require gap locks because they enforce uniqueness strictly. Any attempt to insert a duplicate key will fail immediately, preventing any need for gap locking.

#### Example Scenario

#### Cons
ider a scenario where you have a non-clustered index on OrderID in an Orders table:

SQL
CREATE NONCLUSTERED INDEX idx_OrderID ON Orders(OrderID);

When two concurrent transactions try to insert rows with overlapping OrderID, the database might apply gap locks on the gaps between existing OrderIDs to prevent one transaction from splitting the index.

Detailed Explanation
Insert Operation: When a new row is inserted, the database checks if an index key already exists.
Gap Locks: If the insert operation involves a non-unique index and multiple transactions are inserting overlapping keys, gap locks can be applied to ensure proper ordering and prevent concurrent inserts from causing inconsistencies.
Code Example

Here’s a simplified example in SQL Server:

SQL
-- Create table with a non-clustered index
CREATE TABLE Orders (
    OrderID INT,
    ProductName VARCHAR(100),
    INDEX idx_OrderID NONCLUSTERED (OrderID)
);

-- Insert operations
BEGIN TRANSACTION;
INSERT INTO Orders(OrderID, ProductName) VALUES (101, 'ProductA');
WAITFOR DELAY '00:00:10';
INSERT INTO Orders(OrderID, ProductName) VALUES (102, 'ProductB'); -- This might require gap locks

COMMIT;

#### Common Mistakes
and Follow-Ups
Misunderstanding Gap Locks: Developers often confuse row locks with gap locks.
Ignoring Index Types: Not recognizing that non-unique indexes can lead to additional locking mechanisms during concurrent operations.

#### Closing Summary
The choice of isolation level affects how indexes behave under concurrent inserts, particularly in terms of whether gap locks are applied. Non-unique B-tree index types are more prone to requiring gap locks due to the nature of their key ordering and potential for overlapping keys.

**Code:**

```sql
CREATE TABLE Orders (
    OrderID INT,
    ProductName VARCHAR(100),
    INDEX idx_OrderID NONCLUSTERED (OrderID)
);
```

**Example input / output:**

```text
BEGIN TRANSACTION;
INSERT INTO Orders(OrderID, ProductName) VALUES (101, 'ProductA');
WAITFOR DELAY '00:00:10';
INSERT INTO Orders(OrderID, ProductName) VALUES (102, 'ProductB'); -- This might require gap locks

COMMIT;
```

**Flow:**

1. Begin Transaction
2. Insert OrderID 101
3. Wait for Delay
4. Insert OrderID 102
5. Commit

