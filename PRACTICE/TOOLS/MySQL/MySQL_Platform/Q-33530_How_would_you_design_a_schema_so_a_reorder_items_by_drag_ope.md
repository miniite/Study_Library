---
type: "practice"
difficulty: "Deep"
---

# How would you design a schema so a 'reorder items by drag' operation persists cheaply, avoiding rewriting every row's position on each move?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Deadlocks in MySQL](<../../../../TOOLS/MySQL/MySQL_Platform/Deadlocks_in_MySQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a schema that efficiently handles "reorder items by drag" operations in MySQL without having to rewrite every row’s position on each move, we need to focus on minimizing write operations and ensuring data integrity. The core idea is to use an additional column to track the order of items and update only this column during reorder operations.

#### Triage

The question asks how to efficiently design a schema for handling reorder operations in MySQL without rewriting every row's position each time. This involves understanding the trade-offs between read and write performance, as well as ensuring data consistency. The key is to minimize the number of writes while maintaining an efficient way to query and update the order.

#### First-Principles Solution
Modeling the Schema: We need a table that stores items with an additional column to track their order.
Handling Reorders: Use atomic operations or transactions to ensure consistency during reorder operations.
Query Optimization: Efficiently retrieve and update the order without affecting performance.
Step-by-Step Solution

Define the Table:

Create a table with an id (primary key), name, and order_index.
SQL
CREATE TABLE items (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    order_index INT NOT NULL DEFAULT 0
);

Handling Reorders:

Use a transaction to ensure that the reordering is atomic and consistent.
When an item is moved, update its order_index value without affecting other rows.

#### Example
Code
SQL
-- Start of transaction
START TRANSACTION;

-- Move Item 1 before Item 2 (assuming items are identified by IDs)
UPDATE items SET order_index = @temp := order_index WHERE id = 2;
SET @temp := order_index - 1;
UPDATE items SET order_index = @temp + 1 WHERE id = 1;

-- Commit transaction
COMMIT;

Query Optimization:

Use indexes on the order_index column to optimize queries.
SQL
CREATE INDEX idx_order ON items (order_index);

Handling Edge Cases:

Ensure that the order index does not create a duplicate value by using a unique constraint or sequence generator.

#### Closing Summary
+ Common Mistakes/Follow-Ups

The key to efficient reordering is to use an additional column for tracking the order and ensuring atomic updates through transactions. This approach minimizes write operations, improving performance while maintaining data consistency. Common mistakes include:

Failing to use transactions, leading to inconsistent states.
Overlooking index optimization, which can degrade query performance.

For follow-ups, we might discuss how to scale this solution in a distributed environment or explore alternative methods like using sequence generators for order_index values.

**Code:**

```sql
-- SQL code examples
CREATE TABLE items (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    order_index INT NOT NULL DEFAULT 0
);

START TRANSACTION;
UPDATE items SET order_index = @temp := order_index WHERE id = 2;
SET @temp := order_index - 1;
UPDATE items SET order_index = @temp + 1 WHERE id = 1;
COMMIT;

CREATE INDEX idx_order ON items (order_index);
```

**Example input / output:**

```text
Input:
Initial order of items: [A (0), B (1), C (2)]

Action: Move Item A before Item B

Output:
Final order of items: [A (1), B (2), C (0)]
```

**Data:**

```json
{
    "items": [
        {"id": 1, "name": "Item A", "order_index": 0},
        {"id": 2, "name": "Item B", "order_index": 1},
        {"id": 3, "name": "Item C", "order_index": 2}
    ]
}
```

