---
type: "practice"
difficulty: "Expert"
---

# How do you architect isolation for a two-phase inventory reservation (hold then confirm) so a crash between phases never leaks reserved-but-never-confirmed stock at scale?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Isolation with Read Replicas](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/Isolation_with_Read_Replicas.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When architecting isolation for a two-phase inventory reservation system, ensuring that no reserved-but-never-confirmed stock is leaked requires careful consideration of transaction isolation levels and how they impact the consistency and durability of your database operations. In this scenario, we need to leverage isolation levels like Read Uncommitted (RU), Repeatable Read (RR), and Serializable (S) to prevent data anomalies such as dirty reads, non-repeatable reads, and phantom reads.

At a high level, our goal is to ensure that once an inventory item is reserved, it remains reserved even if another transaction modifies or reads the same record. This requires strong consistency guarantees during both the reservation and confirmation phases. We will use Serializable (S) isolation for critical operations but explore other levels in detail.

**Comparison:**

```
Transaction Isolation Level	Characteristics
Serializable (S)	Ensures no dirty reads, non-repeatable reads, or phantom reads.
Repeatable Read (RR)	Ensures no non-repeatable reads and no phantom reads but allows dirty reads.
Read Committed (RC)	Ensures no dirty reads, but non-repeatable reads are possible.
Read Uncommitted (RU)	Allows all types of data anomalies.
```

**Code:**

```sql
-- Example SQL statements using different isolation levels

-- Using Serializable
BEGIN TRANSACTION;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SELECT * FROM inventory WHERE product_id = 1 FOR UPDATE; -- Reserve the item
-- Perform some operations...
COMMIT;

-- Using Repeatable Read
BEGIN TRANSACTION;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT * FROM inventory WHERE product_id = 1; -- Reserve the item
-- Perform some operations...
COMMIT;
```

**Example input / output:**

```

Example input:

Initial Inventory: {"product_id": 1, "quantity": 50, "reserved_quantity": 0}
Reservation Request: Reserve product with ID 1

Example output (assuming serializable isolation):

JSON
Copy
{
    "transaction": {
        "status": "COMMITTED",
        "message": "Product reserved successfully.",
        "new_state": {
            "product_id": 1,
            "quantity": 49,
            "reserved_quantity": 1
        }
    }
}

Example failure (if a crash occurs between reservation and confirmation):

JSON
Copy
{
    "transaction": {
        "status": "ROLLED_BACK",
        "message": "Product reservation failed due to transaction rollback.",
        "new_state": {
            "product_id": 1,
            "quantity": 50,
            "reserved_quantity": 0
        }
    }
}
```

**Data:**

```json
{
    "inventory": [
        {
            "product_id": 1,
            "quantity": 50,
            "reserved_quantity": 0
        }
    ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. Start
5. Is Reservation Required?
6. Begin Transaction withSerializable Isolation
7. Reserve Product
8. Commit Transaction andConfirm Reservation
9. End of Reservation Phase
10. Is Confirmation Required?
11. Begin Transaction withRepeatable Read Isolation
12. Confirm Reservation
13. Commit Transaction andRelease Locks
14. End of Confirmation Phase

