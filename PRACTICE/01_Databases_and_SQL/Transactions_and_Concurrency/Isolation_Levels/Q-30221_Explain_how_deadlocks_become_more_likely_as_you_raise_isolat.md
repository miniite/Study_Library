---
type: "practice"
difficulty: "Deep"
---

# Explain how deadlocks become more likely as you raise isolation level, using a simple two-transaction example that deadlocks at SERIALIZABLE but not READ COMMITTED.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Lost Updates and Write Skew](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Lost_Updates_and_Write_Skew.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Deadlocks are a common problem in database systems where multiple transactions compete for resources and cannot proceed due to circular waits. Isolation levels determine how transactions see each other's changes, and as we increase isolation, deadlocks become more likely because the system needs to do more work to prevent conflicting operations. Let's explore this through an example using SERIALIZABLE and READ COMMITTED.

In a database environment with READ COMMITTED isolation level, transactions only see committed data, while in SERIALIZABLE, each transaction sees the database as if it were completely isolated from other transactions. This means that SERIALIZABLE enforces serializability, ensuring no conflicts arise due to concurrent operations on the same data.

#### Cons
ider two simple transactions: T1 and T2. Both try to update a table with records A and B. We will walk through how these transactions behave under READ COMMITTED versus SERIALIZABLE isolation levels, leading us to a deadlock in SERIALIZABLE but not in READ COMMITTED.

**Code:**

```sql
CREATE TABLE Records (
    id INT PRIMARY KEY,
    value VARCHAR(255)
);

-- Initial state of the table
INSERT INTO Records (id, value) VALUES (1, 'A'), (2, 'B');
```

**Example input / output:**

```
READ COMMITTED Example

Transaction T1:

SQL
Copy
BEGIN;
SELECT * FROM Records WHERE id = 1 FOR UPDATE; -- Locks row with id=1
UPDATE Records SET value = 'X' WHERE id = 1;   -- Modifies the locked row

Transaction T2:

SQL
Copy
BEGIN;
SELECT * FROM Records WHERE id = 2 FOR UPDATE; -- Locks row with id=2
UPDATE Records SET value = 'Y' WHERE id = 2;   -- Modifies the locked row

In this scenario, both transactions lock and modify different rows. There is no conflict, so no deadlock occurs.

SERIALIZABLE Example

Transaction T1:

SQL
Copy
BEGIN;
SELECT * FROM Records WHERE id IN (1, 2) FOR UPDATE; -- Locks both rows with ids=1 and 2
UPDATE Records SET value = 'X' WHERE id = 1;         -- Modifies the locked row

Transaction T2:

SQL
Copy
BEGIN;
SELECT * FROM Records WHERE id IN (1, 2) FOR UPDATE; -- Tries to lock both rows with ids=1 and 2
-- This locks all records, but transaction T1 has already locked id=1.
-- Transaction T2 cannot proceed until it can acquire the exclusive lock on id=2,
-- which is held by T1. Meanwhile, T1 waits for the release of the lock on id=2.
UPDATE Records SET value = 'Y' WHERE id = 2;         -- This transaction will wait indefinitely

In this case, T1 locks both rows first and modifies one, while T2 tries to acquire a lock on all records but is blocked by T1’s hold. The transactions are now in a circular wait state, leading to a deadlock.
```

**Flow:**

1. Transaction T1 starts
2. Locks row 1 and modifies
3. Transaction T2 starts
4. Tries to lock row 1 and wait
5. Transaction T2 modifies row 2
6. Transaction T1 tries to lock row2 and waits
7. Deadlock detected,transactions hang

