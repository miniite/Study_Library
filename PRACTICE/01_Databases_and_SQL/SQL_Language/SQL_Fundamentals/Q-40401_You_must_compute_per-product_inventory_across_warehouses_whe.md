---
type: "practice"
difficulty: "Deep"
---

# You must compute per-product inventory across warehouses where some warehouses report negative adjustments. Walk through why SUM over a JOIN can double-count and how a pre-aggregated CTE per warehouse fixes it.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Join Fan-out](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Join_Fan-out.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When computing the total product inventory across multiple warehouses, you may encounter issues if some warehouses report negative adjustments (which effectively mean products have been removed or are lost). A common approach is to use a JOIN combined with a SUM, but this can lead to double-counting. To solve this problem, we can pre-aggregate the data per warehouse and then sum these aggregated values correctly.

#### Triaging the Question

The question revolves around understanding how SQL queries can introduce counting errors when dealing with negative adjustments in inventory reporting. The key points are:

Double-Counting Issue: Using a JOIN combined with a SUM can result in items being counted multiple times.
Correct Aggregation: Pre-aggregating the data per warehouse ensures accurate totals without double-counting.

#### Solution
Walkthrough
Initial Attempt: Join and Sum

#### Cons
ider a typical schema where we have two tables: products (containing product details) and inventory_adjustments (log of changes in inventory).

SQL
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name VARCHAR(255)
);

CREATE TABLE inventory_adjustments (
    id SERIAL PRIMARY KEY,
    warehouse_id INT,
    product_id INT,
    adjustment_amount DECIMAL,
    timestamp TIMESTAMP
);

A naive approach might look like this:

SQL
SELECT p.product_id, SUM(i.adjustment_amount) AS total_inventory
FROM products p
JOIN inventory_adjustments i ON p.product_id = i.product_id
GROUP BY p.product_id;

This query will sum the adjustment amounts directly. However, it can double-count adjustments that are negative.

Correct Approach: Pre-Aggregate with CTE

To avoid this issue, we can pre-aggregate the inventory_adjustments data by warehouse and then join these aggregated values to the products.

SQL
WITH warehouse_adjustments AS (
    SELECT product_id, SUM(adjustment_amount) AS total_adjustment
    FROM inventory_adjustments
    GROUP BY product_id, warehouse_id
)
SELECT p.product_id, COALESCE(SUM(wa.total_adjustment), 0) AS total_inventory
FROM products p
LEFT JOIN warehouse_adjustments wa ON p.product_id = wa.product_id
GROUP BY p.product_id;

In this query:

CTE warehouse_adjustments: Aggregates the adjustment amounts for each product in each warehouse.
Main Query: Joins these pre-aggregated values with the products table, using a LEFT JOIN to ensure all products are included even if there are no adjustments.

#### Closing Summary

By pre-aggregating the data per warehouse and then summing those aggregated values, we avoid double-counting issues that can arise from directly joining and summing the raw adjustment records. This approach ensures that each product's inventory is accurately represented across all warehouses.

#### Common Mistakes/Follow-Ups
Mistake: Failing to pre-aggregate data before joining.

#### Follow-Up
Discuss edge cases like products with no adjustments, handling NULL values in the final aggregate, and performance considerations for large datasets.

**Comparison:**

| Product ID | Total Inventory |
|-----------|-----------------|
| 1         | 12              |
| 2         | 8               |

**Code:**

```sql
WITH warehouse_adjustments AS (
    SELECT product_id, SUM(adjustment_amount) AS total_adjustment
    FROM inventory_adjustments
    GROUP BY product_id, warehouse_id
)
SELECT p.product_id, COALESCE(SUM(wa.total_adjustment), 0) AS total_inventory
FROM products p
LEFT JOIN warehouse_adjustments wa ON p.product_id = wa.product_id
GROUP BY p.product_id;
```

**Example input / output:**

```text
WITH warehouse_adjustments AS (
    SELECT product_id, SUM(adjustment_amount) AS total_adjustment
    FROM inventory_adjustments
    GROUP BY product_id, warehouse_id
)
SELECT p.product_id, COALESCE(SUM(wa.total_adjustment), 0) AS total_inventory
FROM products p
LEFT JOIN warehouse_adjustments wa ON p.product_id = wa.product_id
GROUP BY p.product_id;

-- Expected Output:
-- +------------+----------------+
-- | product_id | total_inventory |
-- +------------+----------------+
-- |          1 |              12 |
-- |          2 |               8 |
-- +------------+----------------+
```

**Data:**

```sql
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name VARCHAR(255)
);

INSERT INTO products (product_id, name) VALUES
(1, 'Laptop'),
(2, 'Smartphone');

CREATE TABLE inventory_adjustments (
    id SERIAL PRIMARY KEY,
    warehouse_id INT,
    product_id INT,
    adjustment_amount DECIMAL,
    timestamp TIMESTAMP
);

INSERT INTO inventory_adjustments (warehouse_id, product_id, adjustment_amount) VALUES
(1, 1, 10),
(2, 1, -5),
(3, 1, 7),
(4, 2, 8);
```

**Flow:**

1. Direct Aggregation
2. Pre-aggregation
3. Yes
4. No
5. Products Table
6. Has Adjustments?
7. Pre-Aggregate by Warehouse
8. Join with Products on ProductID
9. Sum Total Adjustments
10. Direct Join and Sum
11. Final Inventory Calculation
12. Final Inventory Calculation(May Double-Count)

