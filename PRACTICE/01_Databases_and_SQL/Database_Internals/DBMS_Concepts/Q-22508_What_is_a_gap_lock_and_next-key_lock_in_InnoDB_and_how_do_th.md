---
type: "practice"
difficulty: "Expert"
---

# What is a gap lock and next-key lock in InnoDB and how do they prevent phantoms under REPEATABLE READ while sometimes causing surprising deadlocks?

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ Lock Granularity](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Lock_Granularity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Gap locks and next-key locks are critical concepts in understanding how InnoDB manages concurrency, especially within the context of the REPEATABLE READ isolation level. These locking mechanisms help prevent phantom reads but can sometimes lead to unexpected deadlocks. Let’s break down these concepts step by step.

First, a gap lock is used when a transaction attempts to insert a new row in a specific range, where no rows currently exist. A gap lock covers the entire gap between two existing index keys or the start/end of an index if there are no other keys nearby. Meanwhile, a next-key lock combines an index record lock with a gap lock. It locks the exact index key and any gaps before it.

In REPEATABLE READ isolation, InnoDB uses these locks to ensure that once you read a row, it will not disappear or change by another transaction. However, next-key locks can sometimes lead to deadlocks because they lock both data records and gaps between them. This can cause transactions to wait indefinitely for the same resources held by others.

Now, let's explore how these locks prevent phantom reads and what can go wrong with deadlocks.

**Code:**

```sql
-- Example of next-key lock in REPEATABLE READ
START TRANSACTION;
SELECT * FROM orders WHERE order_id > 10 FOR UPDATE; -- This will acquire a next-key lock on the row where order_id = 11 and gaps before it.
```

**Example input / output:**

```text
-- Input: Transaction T1 starts with a REPEATABLE READ isolation level.
START TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Output:
-- InnoDB acquires next-key locks on the rows and gaps as specified in the query.
SELECT * FROM orders WHERE order_id > 2; -- Locks rows where order_id = 3 and the gap before it.

-- Input: Another transaction T2 starts and tries to insert a new row at order_id 4.
START TRANSACTION;
INSERT INTO orders (order_id, customer_name) VALUES (4, 'David');

-- Output:
-- InnoDB waits for the next-key lock held by T1 on the gap before order_id = 3, causing deadlock if not resolved.

-- Input: Transaction T1 then tries to update an existing row.
UPDATE orders SET customer_name = 'Updated' WHERE order_id = 2;

-- Output:
-- This may conflict with another transaction trying to insert a new row in the same range, leading to a lock wait or deadlock.
```

**Data:**

```json
{
    "table": {
        "name": "orders",
        "columns": ["order_id", "customer_name"],
        "rows": [
            {"order_id": 1, "customer_name": "Alice"},
            {"order_id": 2, "customer_name": "Bob"},
            {"order_id": 3, "customer_name": "Charlie"}
        ]
    }
}
```

