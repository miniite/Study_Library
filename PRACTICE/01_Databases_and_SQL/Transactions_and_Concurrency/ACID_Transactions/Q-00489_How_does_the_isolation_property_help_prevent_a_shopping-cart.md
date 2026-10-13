---
type: "practice"
difficulty: "Intermediate"
---

# How does the isolation property help prevent a shopping-cart double-purchase problem?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Isolation Levels in Transactions](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Isolation_Levels_in_Transactions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a database context, ACID properties ensure that transactions are processed reliably and consistently. The isolation property specifically prevents concurrency issues, such as dirty reads, non-repeatable reads, and phantom reads. For a shopping-cart double-purchase problem, the isolation property ensures that multiple transactions do not interfere with each other, effectively preventing a customer from accidentally placing the same item in their cart twice or purchasing an item more than once.

To understand this better, let's break down the isolation levels: read uncommitted (RU), read committed (RC), repeatable read (RR), and serializable (S). The shopping-cart double-purchase problem can be mitigated at different isolation levels, with serializable being the most stringent but also the safest in preventing such issues.

**Comparison:**

```
table "Cart" {
    user_id Integer
    product_id String
    quantity Integer
}
```

**Code:**

```sql
-- Example of a transaction that might cause a double purchase without proper isolation
START TRANSACTION;
SELECT product_id, quantity FROM cart WHERE user_id = 12345 AND product_id = 'Laptop';
-- Assume product_id is found and updated in the next statement
UPDATE cart SET quantity = quantity - 1 WHERE user_id = 12345 AND product_id = 'Laptop';

COMMIT;
```

**Example input / output:**

```text
Given an initial state of the cart with `user_id = 12345` and `product_id = 'Laptop'`, quantity set to 2, we simulate a transaction that decrements the quantity by 1. If this transaction is not properly isolated, another concurrent transaction could read the same data, decrement it again, and commit before the first transaction can finalize its update.
```

**Data:**

```json
{
  "user_id": 12345,
  "cart_items": [
    {
      "product_id": "Laptop",
      "quantity": 2
    }
  ]
}
```

**Flow:**

1. Transaction 1 Starts
2. Read Cart Items
3. Decrement Quantity
4. Commit Transaction
5. Transaction 2 Starts
6. Read Cart Items (Same Data)
7. Decrement Quantity (Again)
8. Commit Transaction

