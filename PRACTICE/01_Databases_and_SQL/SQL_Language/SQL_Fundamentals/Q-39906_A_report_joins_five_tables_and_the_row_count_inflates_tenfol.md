---
type: "practice"
difficulty: "Intermediate"
---

# A report joins five tables and the row count inflates tenfold. Walk me through diagnosing the fan-out and fixing the double-counting with pre-aggregation or DISTINCT-aware sums.

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Join Fan-out](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Join_Fan-out.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To diagnose why a SQL query joining five tables results in an inflated row count, we need to identify the source of the "fan-out," which is likely due to duplicate rows being counted multiple times across the joins. The core solution involves either pre-aggregation or using DISTINCT-aware sums to ensure that each row is counted only once.

First, let's reframe the question and assumptions:

We are dealing with a query that joins five tables.
The result set contains more rows than expected, indicating potential double-counting.
The goal is to identify the cause of this inflation and fix it by either pre-aggregating or using DISTINCT-aware sums.
Step-by-Step Solution

Understand the Joins: Start by examining each join condition in the query to see if any pairs of tables could lead to duplicate rows. For example, a common issue is when a join on an equality column like id fails to handle nulls or duplicates appropriately.

Identify the Fan-Out Source:

Check for self-joins and joins between tables with high cardinality columns.
Look at the data distribution of each table, especially in columns used in join conditions.

Use DISTINCT: If the fan-out is due to duplicate rows within a single table or across multiple tables, wrap your query in a SELECT DISTINCT ... FROM (subquery). This ensures that each row is counted only once, even if it appears multiple times in intermediate results.

Pre-Aggregation:

Group data before joining by aggregating columns from other tables.
For instance, if you have a large table with millions of rows and want to join it with smaller lookup tables, first group the large table on key columns and then perform the join.

#### Example
Query

Assume we have five tables: orders, customers, products, suppliers, and locations. The query might look like this:

SQL
SELECT o.order_id, c.customer_name, p.product_name, s.supplier_name, l.location_name
FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN products p ON o.product_id = p.id
JOIN suppliers s ON p.supplier_id = s.id
JOIN locations l ON o.location_id = l.id;

To diagnose and fix the fan-out:

Check for Self-Joins: Ensure there are no self-joins that could create duplicates.

Identify Potential Overlaps:

Look at the join conditions: customer_id, product_id, supplier_id, and location_id.
Check if any of these columns can have null values or multiple entries.

Test with Smaller Datasets: If possible, test your query on a subset of data to see where the fan-out occurs.

Fixing with DISTINCT-Aware Sums

If the problem is due to double-counting within tables:

SQL
SELECT o.order_id, c.customer_name, p.product_name, s.supplier_name, l.location_name,
       COUNT(DISTINCT o.order_id) AS order_count,
       SUM(CASE WHEN o.order_id = DISTINCT o.order_id THEN 1 ELSE 0 END) AS distinct_order_count
FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN products p ON o.product_id = p.id
JOIN suppliers s ON p.supplier_id = s.id
JOIN locations l ON o.location_id = l.id
GROUP BY o.order_id, c.customer_name, p.product_name, s.supplier_name, l.location_name;

This query ensures that each order is counted only once in the final result set.

#### Closing Summary

By methodically diagnosing and addressing potential sources of fan-out, we can ensure our SQL queries are efficient and accurate. Using DISTINCT-aware sums or pre-aggregating data before joining tables helps mitigate double-counting issues.

#### Common Mistakes

Failing to recognize self-joins as a source of fan-out.
Not properly handling null values in join conditions.
Overlooking the importance of distinct counts, especially when dealing with large datasets.

**Comparison:**

```
Orders
Customers
Products
Suppliers
Locations
```

**Code:**

```sql
-- Example query using DISTINCT and pre-aggregation

WITH aggregated_orders AS (
    SELECT order_id, customer_id, product_id, supplier_id, location_id,
           COUNT(*) OVER (PARTITION BY order_id) as order_count
    FROM orders o
)

SELECT ao.order_id, c.customer_name, p.product_name, s.supplier_name, l.location_name,
       ao.order_count
FROM aggregated_orders ao
JOIN customers c ON ao.customer_id = c.id
JOIN products p ON ao.product_id = p.id
JOIN suppliers s ON p.supplier_id = s.id
JOIN locations l ON ao.location_id = l.id;
```

**Example input / output:**

```text
-- Input (sample data)

WITH aggregated_orders AS (
    SELECT order_id, customer_id, product_id, supplier_id, location_id,
           COUNT(*) OVER (PARTITION BY order_id) as order_count
    FROM orders o
)

SELECT ao.order_id, c.customer_name, p.product_name, s.supplier_name, l.location_name,
       ao.order_count
FROM aggregated_orders ao
JOIN customers c ON ao.customer_id = c.id
JOIN products p ON ao.product_id = p.id
JOIN suppliers s ON p.supplier_id = s.id
JOIN locations l ON ao.location_id = l.id;

-- Output

+------------+----------------+--------------+---------------+-------------+------------+
| order_id   | customer_name  | product_name | supplier_name | location_name| order_count |
+------------+----------------+--------------+---------------+-------------+------------+
| 1          | Alice          | Widget       | TechCorp      | NYC Office  | 2          |
+------------+----------------+--------------+---------------+-------------+------------+
```

**Data:**

```json
{
  "orders": [
    {"order_id": 1, "customer_id": 101, "product_id": 201, "supplier_id": 301, "location_id": 401},
    {"order_id": 2, "customer_id": 101, "product_id": 201, "supplier_id": 301, "location_id": 401}
  ],
  "customers": [{"id": 101, "name": "Alice"}],
  "products": [{"id": 201, "name": "Widget", "supplier_id": 301}],
  "suppliers": [{"id": 301, "name": "TechCorp"}],
  "locations": [{"id": 401, "name": "NYC Office"}]
}
```

**Flow:**

1. Orders
2. order_id = customer_id
3. Order_id
4. Customer
5. Product_id
6. Product
7. Supplier_id
8. Supplier
9. Location_id
10. Location

