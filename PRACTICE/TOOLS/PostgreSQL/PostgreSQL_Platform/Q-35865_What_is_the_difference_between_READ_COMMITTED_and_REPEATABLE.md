---
type: "practice"
difficulty: "Deep"
---

# What is the difference between READ COMMITTED and REPEATABLE READ behavior for a query run twice in the same transaction?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Deadlocks and Lock Ordering](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Deadlocks_and_Lock_Ordering.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In PostgreSQL, READ COMMITTED and REPEATABLE READ are isolation levels that determine how transactions handle dirty reads, non-repeatable reads, and phantom reads. When a query is run twice within the same transaction under different isolation levels, it will produce different results due to these differences.

#### Triage

The question asks for the difference between READ COMMITTED and REPEATABLE READ when a query is executed multiple times in the same transaction. The key points to consider are:

Transaction Context: Both queries run within the same transaction.
Isolation Levels: Understand how each isolation level affects visibility of data.

#### First Principles Solution
READ COMMITTED Behavior

In READ COMMITTED mode, a transaction sees only committed data. If one transaction modifies a row, it will not be visible to another transaction until the modifying transaction commits. Here’s what happens when running a query twice in the same transaction:

The first execution of the query at the start of the transaction reads the initial state.
If a transaction modifies some rows between these two executions, those changes are not seen by the second execution.
REPEATABLE READ Behavior

In REPEATABLE READ mode, once data is read, it appears constant within the same transaction until that transaction ends. Any modifications made by other transactions are not visible to this transaction while the current query is running:

The first execution of the query reads a snapshot of the database state.
Even if another transaction modifies some rows between these two executions, those changes are not seen in the second execution.

#### Closing Summary
and Common Mistakes/Follow-Ups
Common Mistake: Assuming that REPEATABLE READ will see changes made by other transactions since it allows reading multiple times within a single transaction. However, such changes are only visible after the transaction commits.

#### Follow-Up
Discuss how to handle conflicts in multi-user environments and strategies for maintaining data consistency.

#### Example Scenario

#### Cons
ider a scenario where two transactions modify the same row:

SQL
BEGIN;
SELECT * FROM orders WHERE order_id = 1; -- First execution of query at isolation level READ COMMITTED
-- Transaction 2 modifies the row: UPDATE orders SET status = 'shipped' WHERE order_id = 1;
SELECT * FROM orders WHERE order_id = 1; -- Second execution of query, but no change in state due to isolation level
COMMIT;

In READ COMMITTED, the second execution might see the updated state. In REPEATABLE READ, it would not.

**Comparison:**

| Isolation Level | Behavior                           |
|-----------------|-----------------------------------|
| READ COMMITTED  | Sees only committed data          |
| REPEATABLE READ | Consistent snapshot within the tx |

**Code:**

```sql
-- Example SQL statements for illustration
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;
SELECT * FROM orders WHERE order_id = 1;
UPDATE orders SET status = 'shipped' WHERE order_id = 1;
SELECT * FROM orders WHERE order_id = 1; -- Different result here due to isolation level
COMMIT;

BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT * FROM orders WHERE order_id = 1;
UPDATE orders SET status = 'shipped' WHERE order_id = 1;
SELECT * FROM orders WHERE order_id = 1; -- Same result as the first select, no change due to isolation level
COMMIT;
```

**Example input / output:**

```text
- **Input**: `BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED; SELECT * FROM orders WHERE order_id = 1; UPDATE orders SET status = 'shipped' WHERE order_id = 1; SELECT * FROM orders WHERE order_id = 1; COMMIT;`
- **Output**:
  - First Select: `{ "order_id": 1, "status": "pending" }`
  - Second Select: `{ "order_id": 1, "status": "shipped" }` (in `READ COMMITTED`)

- **Input**: `BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ; SELECT * FROM orders WHERE order_id = 1; UPDATE orders SET status = 'shipped' WHERE order_id = 1; SELECT * FROM orders WHERE order_id = 1; COMMIT;`
- **Output**:
  - First Select: `{ "order_id": 1, "status": "pending" }`
  - Second Select: `{ "order_id": 1, "status": "pending" }` (no change due to isolation level)
```

**Data:**

```json
{
  "orders": [
    {
      "order_id": 1,
      "status": "pending"
    }
  ]
}
```

**Flow:**

1. Start Transaction
2. Isolation Level?
3. READ COMMITTED
4. REPEATABLE READ
5. First SELECT - Reads InitialState
6. UPDATE - By Other Txn
7. Second SELECT - Sees UpdatedState if changes seen
8. First SELECT - Snapshot atStart
9. UPDATE - By Other Txn
10. Second SELECT - No Change inState

