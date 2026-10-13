---
type: "practice"
difficulty: "Deep"
---

# Explain Spanner's interleaved tables and how they physically co-locate child rows with parents; when does interleaving materially improve latency, and what are the migration hazards if hierarchy changes later?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Distributed Query Execution](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/NewSQL/Distributed_Query_Execution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Interleaved tables in Google Cloud Spanner allow parent-child relationships to be physically co-located within a single table, which can significantly reduce latency when performing queries that traverse these relationships. This feature is particularly useful in scenarios where there's a strong hierarchical structure between entities, as it minimizes the need for expensive joins across separate tables.

Spanner's interleaved tables work by storing child records under their parent record’s key path. For example, if you have an order and its items, each item record would be stored with the order's primary key as part of its own key path, ensuring that all related data is kept together. This physical co-location means that when a query retrieves an order, it can also fetch its associated items without having to perform additional read operations.

However, while interleaving offers significant performance benefits for certain queries, it comes with trade-offs and migration challenges if the database schema or business logic evolves over time. Let's break down how interleaved tables function, their advantages, disadvantages, and potential pitfalls.

**Comparison:**

| OrderId | CustomerId | Status   |
|---------|------------|----------|
| 12345   | 98765      | Shipped  |
| ItemId | OrderId | Price | Quantity |
|--------|---------|-------|----------|
| 101    | 12345   | 29.99 | 2        |
| 102    | 12345   | 49.99 | 1        |

**Code:**

```sql
-- Example of defining an interleaved table in Spanner
CREATE TABLE Orders (
  OrderId INT64,
  CustomerId INT64,
  Status STRING(10),
  -- Other order fields...
) PRIMARY KEY (OrderId);

CREATE TABLE Items (
  ItemId INT64,
  OrderId INT64,
  Price DECIMAL(38, 9),
  Quantity INT64,
  -- Other item fields...
) INTERLEAVE IN PARENT Orders ON DELETE CASCADE;
```

**Example input / output:**

```text
Input:
- SELECT * FROM Orders WHERE OrderId = 12345;

Output:
{
  "OrderId": 12345,
  "CustomerId": 98765,
  "Status": "Shipped",
  "Items": [
    { "ItemId": 101, "Price": 29.99, "Quantity": 2 },
    { "ItemId": 102, "Price": 49.99, "Quantity": 1 }
  ]
}
```

**Data:**

```json
{
  "OrderId": 12345,
  "CustomerId": 98765,
  "Status": "Shipped",
  "Items": [
    { "ItemId": 101, "Price": 29.99, "Quantity": 2 },
    { "ItemId": 102, "Price": 49.99, "Quantity": 1 }
  ]
}
```

**Flow:**

1. Yes
2. No
3. User Query
4. Interleaved Table Lookup
5. Retrieve Parent and Child Data
6. Join Across Tables
7. Optimized Performance
8. Larger Latency

