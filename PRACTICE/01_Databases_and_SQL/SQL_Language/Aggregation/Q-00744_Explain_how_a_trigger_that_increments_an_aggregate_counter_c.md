---
type: "practice"
difficulty: "Deep"
---

# Explain how a trigger that increments an aggregate counter can become a hot-row contention bottleneck, and how you would shard or batch it.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Advanced Aggregation](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/Advanced_Aggregation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a database system, especially one with high transaction volumes, we must be mindful of potential bottlenecks that could arise from certain operations, such as using aggregate functions in triggers to increment counters. In this context, let's consider a trigger that increments an aggregate counter whenever a row is inserted or updated. This operation can lead to hot-row contention due to the frequent reads and writes it incurs on the same row.

To understand why this happens, imagine a scenario where you have a products table with an orders_count column that tracks how many orders each product has received. A trigger increments this count whenever a new order is placed for that product. If multiple transactions are trying to insert or update these counters simultaneously, they will often contend on the same row, causing locks and delays.

To mitigate this hot-row contention, we can consider sharding or batching the counter updates. Let's explore both approaches in detail.

**Code:**

```sql
-- Example of a trigger that increments orders_count
CREATE TRIGGER increment_orders_count
AFTER INSERT ON orders
FOR EACH ROW
BEGIN
    UPDATE products SET orders_count = orders_count + 1 WHERE id = NEW.product_id;
END;
```

**Example input / output:**

```

Given the input data:

JSON
Copy
{
  "orders": [
    { "order_id": 105, "product_id": 1 },
    { "order_id": 106, "product_id": 2 },
    { "order_id": 107, "product_id": 1 }
  ]
}

The expected output would be:

JSON
Copy
{
  "products": [
    { "id": 1, "name": "Product A", "orders_count": 53 },
    { "id": 2, "name": "Product B", "orders_count": 32 }
  ]
}
```

**Data:**

```json
{
  "products": [
    { "id": 1, "name": "Product A", "orders_count": 50 },
    { "id": 2, "name": "Product B", "orders_count": 30 }
  ],
  "orders": [
    { "order_id": 101, "product_id": 1 },
    { "order_id": 102, "product_id": 2 },
    { "order_id": 103, "product_id": 1 },
    { "order_id": 104, "product_id": 2 }
  ]
}
```

