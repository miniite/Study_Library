---
type: "practice"
difficulty: "Deep"
---

# How do phantom reads specifically affect aggregate constraints like 'no more than N rows matching a predicate'? Why can't row locks alone stop them?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Phantoms and Double Booking](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Phantoms_and_Double_Booking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Phantom reads are a type of read skew that occur when a transaction reads new data that was created by another transaction after it started but before it finishes. This is particularly problematic with aggregate constraints such as 'no more than N rows matching a predicate,' because these constraints rely on the number of rows that match certain conditions, which can change unexpectedly due to phantom reads.

To understand why row locks alone cannot prevent phantom reads from affecting aggregate constraints, we need to delve into how isolation levels and transactional consistency are managed in database systems. Let's break this down step-by-step:

Understanding Phantom Reads:

A phantom read occurs when a transaction sees new rows that did not exist at the start of another transaction.
For example, if you run SELECT COUNT(*) FROM table WHERE column = 'some_value', and during your query execution, a new row matching this predicate is inserted by another transaction, your count will be off.

Row Locks vs Phantom Reads:

Row locks (RU isolation level) prevent other transactions from modifying or reading locked rows.
However, they do not protect against phantom reads because the locking mechanism does not cover the gap between when a row is inserted and read.

Why Row Locks Aren't Enough for Aggregate Constraints:

#### Cons
ider a scenario where you have an aggregate constraint that limits the number of rows with status = 'active' to 10.
If another transaction inserts a new row with status = 'active', and this insert happens after your initial read but before you commit, the count will increase unexpectedly.

#### Solution
Serialization (Serializable Isolation Level):

The Serializable isolation level ensures that transactions are executed in a serial manner, meaning they are isolated from each other as if they were run one at a time.
This prevents phantom reads by ensuring that any inserted rows are visible only after the transaction that inserted them has committed.

#### Trade-offs
of Serialization:

While Serializable is powerful and prevents all types of non-repeatable reads, it can also lead to increased contention and performance issues due to its high overhead.
It's generally used in critical transactions where data integrity must be absolutely guaranteed but should not be the default setting for every transaction.

Implementing Serialization:

To enforce Serializable isolation level, you would use SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;.
This ensures that your transaction is run as if it were the only one in the system, thus preventing phantom reads and maintaining aggregate constraints accurately.

#### Common Mistakes
and Follow-Ups:

Assuming row locks are sufficient to handle all types of read skews.
Not understanding the importance of consistency levels in complex data models and transactions.

**Comparison:**

```
Before Insert	After Insert
Count: 1	Count: 2
```

**Code:**

```sql
-- Example of setting isolation level for a transaction
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
BEGIN;

SELECT COUNT(*) FROM orders WHERE status = 'active';
INSERT INTO orders (customer_id, product_id, status) VALUES (1, 201, 'active');

COMMIT;
```

**Example input / output:**

```text
-- Initial state of the table
SELECT COUNT(*) FROM orders WHERE status = 'active';

-- New row inserted by another transaction
INSERT INTO orders (customer_id, product_id, status) VALUES (1, 201, 'active');

-- Final count after the insert
SELECT COUNT(*) FROM orders WHERE status = 'active';
```

**Data:**

```json
{
  "table": {
    "id": "orders",
    "columns": [
      { "name": "order_id", "type": "integer" },
      { "name": "customer_id", "type": "integer" },
      { "name": "product_id", "type": "integer" },
      { "name": "status", "type": "string" }
    ],
    "data": [
      { "order_id": 1, "customer_id": 1, "product_id": 200, "status": "active" },
      { "order_id": 2, "customer_id": 1, "product_id": 201, "status": "inactive" }
    ]
  }
}
```

**Flow:**

1. Serializable
2. RU/RC
3. Transaction starts
4. Isolation Level?
5. Enforce aggregate constraints
6. Row locks preventmodifications but not newrows
7. Phantom reads are prevented
8. Aggregate constraints violateddue to phantom reads

