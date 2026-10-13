---
type: "practice"
difficulty: "Intermediate"
---

# What happens to isolation guarantees when you mix autocommit statements with an explicit transaction in the same session? Give a bug this causes.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Lost Updates and Write Skew](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Lost_Updates_and_Write_Skew.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with database systems, understanding how different isolation levels and transaction management interact is crucial to maintaining data consistency and integrity. In particular, mixing autocommit statements with an explicit transaction within the same session can lead to unexpected behavior in terms of isolation guarantees. This answer will explore this scenario, provide a concrete example, and discuss potential bugs that might arise.

TL;DR

Mixing autocommit statements with an explicit transaction within the same session can result in varying levels of isolation guarantees depending on the database system's implementation. Specifically, unexpected interleaving of transactions may occur, leading to issues like dirty reads or non-repeatable reads. This situation is often exacerbated by default settings and can be problematic for multi-threaded applications.

#### Triage

The question probes an understanding of how transaction management interacts with autocommit behavior in the context of isolation levels (RU/RC/RR/Serializable). The key assumptions are that we're dealing with a database system supporting these isolation levels, such as PostgreSQL or MySQL. We'll explore this scenario step-by-step from first principles.

Step-By-Step Solution

Understanding Autocommit and Transactions:

autocommit mode is typically enabled by default in many databases, where each SQL statement is treated as a separate transaction.
Explicit transactions are started using the BEGIN statement, allowing for multiple statements within the same transaction.

Isolation Levels (RU/RC/RR/Serializable):

Read Uncommitted (RU): Allows reading of uncommitted data.
Read Committed (RC): Only committed data can be read.
Repeatable Read (RR): Ensures that once a value is retrieved, it cannot change until the transaction completes.
Serializable: Ensures full isolation by locking all accessed rows.

Mixing Autocommit with Explicit Transactions:

When autocommit is enabled, each statement is committed immediately after execution. If an explicit transaction is started within the same session, statements inside this transaction will be managed differently from those outside.
The exact behavior depends on the database system’s implementation and configuration.

#### Example Scenario
Let's consider a simplified example using PostgreSQL with autocommit enabled:

SQL
-- Session 1: Autocommit Mode
INSERT INTO items (id, name) VALUES (1, 'Item1');

-- Session 2: Explicit Transaction Mode
BEGIN;
SELECT id FROM items WHERE id = 1; -- No result as the item is not committed yet.
COMMIT;

-- Session 1: Another Insert
INSERT INTO items (id, name) VALUES (2, 'Item2');

In this scenario:

The first INSERT in Session 1 occurs under autocommit mode and gets immediately committed.
The second INSERT also occurs under autocommit mode but is not visible to the explicit transaction started in Session 2 until it commits.
Potential Bugs:
If an explicit transaction starts after a statement has been executed under autocommit, the transaction might see data that was updated by other transactions running under autocommit.
For instance, if Session 2 starts its transaction before the first INSERT in Session 1 is committed, it won't see this new row. Conversely, if Session 2 commits after the second INSERT, it will see both rows.

#### Closing Summary
and Common Mistakes

Mixing autocommit statements with explicit transactions within the same session can lead to inconsistent isolation guarantees because of how data visibility changes between different transaction modes. Developers must be aware that this can result in issues like dirty reads or non-repeatable reads, especially if they rely on specific isolation levels for their application's correctness.

#### Common Mistakes
and Follow-Ups
Not Understanding Autocommit Implications: Assuming all statements are part of a single implicit transaction.
Ignoring Transaction Boundaries: Failing to recognize the differences in behavior between autocommit and explicit transactions.
Incorrect Isolation Level Configuration: Using isolation levels without considering how they interact with autocommit settings.

To mitigate these issues, developers should carefully manage their sessions and explicitly control transaction boundaries where necessary.

**Code:**

```sql
-- Example SQL Statements for Autocommit Mode
INSERT INTO items (id, name) VALUES (1, 'Item1');
INSERT INTO items (id, name) VALUES (2, 'Item2');

-- Example SQL Statements for Explicit Transaction Mode
BEGIN;
SELECT id FROM items WHERE id = 1; -- No result as the item is not committed yet.
COMMIT;
```

**Example input / output:**

```

Session 1:

SQL
Copy
-- Autocommit Mode
INSERT INTO items (id, name) VALUES (1, 'Item1');
INSERT INTO items (id, name) VALUES (2, 'Item2');

Session 2:

SQL
Copy
-- Explicit Transaction Mode
BEGIN;
SELECT id FROM items WHERE id = 1; -- No result as the item is not committed yet.
COMMIT;

Expected Output in Session 2 after Commit in Session 1:

SELECT * FROM items should return both rows (1, 'Item1') and (2, 'Item2').
```

**Data:**

```json
{
  "items": [
    {"id": 1, "name": "Item1"},
    {"id": 2, "name": "Item2"}
  ]
}
```

