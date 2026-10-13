---
type: "practice"
difficulty: "Expert"
---

# When a query joins to two child tables and then SUMs, why do you get a fan-out cartesian inflation, and how do you fix it with pre-aggregation before the join?

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ Advanced Aggregation](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/Advanced_Aggregation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle this database developer question, we need to understand what happens when a query joins two child tables and then applies an aggregate function like SUM after the join. This can lead to a Cartesian product, or "fan-out," where each row from one table is joined with every row from the other table, causing a significant increase in the number of rows processed. We'll explore why this happens and how we can mitigate it through pre-aggregation.

Firstly, let's break down the query structure. Suppose you have three tables: orders, products, and categories. The query might join orders with both products and categories to calculate a sum of order amounts grouped by product or category. If there are 100 orders, 50 products, and 25 categories, the initial join would result in (100 \times 50 \times 25 = 125,000) rows, leading to a massive data volume.

Now, let's dive into why this happens. When you perform an INNER JOIN between orders and products, for each order row, the database retrieves all corresponding product rows. Similarly, joining with categories then results in every product row being joined with every category row, leading to a fan-out effect. This is because the aggregate function (SUM) operates on this inflated dataset.

To mitigate this issue, we can pre-aggregate the data by grouping and aggregating before performing the join. For example, we could first sum up the order amounts for each product or category in separate queries and then join these aggregated results with orders. This reduces the number of rows being processed significantly, improving performance.

**Comparison:**

| Initial Query Results | Pre-aggregation Approach Results |
| --- | --- |
| Order ID, Product Name, Category Name, Total Sales (125,000 rows) | Total Sales, Product Name, Category Name (1,250 rows) |

**Code:**

```sql
-- Example of a pre-aggregation approach

-- Step 1: Summarize orders by products
WITH ProductSums AS (
    SELECT product_id, SUM(order_amount) as total_sales
    FROM orders
    GROUP BY product_id
)

-- Step 2: Join with categories and calculate final result
SELECT p.product_name, c.category_name, ps.total_sales
FROM ProductSums ps
JOIN products p ON ps.product_id = p.id
JOIN categories c ON p.category_id = c.id;
```

**Example input / output:**

```text
-- Input: orders table with 100 rows, products table with 50 rows, categories table with 25 rows

-- Initial query without pre-aggregation:
SELECT o.order_id, p.product_name, c.category_name, SUM(o.order_amount) as total_sales
FROM orders o
JOIN products p ON o.product_id = p.id
JOIN categories c ON p.category_id = c.id
GROUP BY o.order_id, p.product_name, c.category_name;

-- Output: 125,000 rows (inflated dataset)

-- Pre-aggregation approach:
WITH ProductSums AS (
    SELECT product_id, SUM(order_amount) as total_sales
    FROM orders
    GROUP BY product_id
)
SELECT ps.total_sales, p.product_name, c.category_name
FROM ProductSums ps
JOIN products p ON ps.product_id = p.id
JOIN categories c ON p.category_id = c.id;

-- Output: 50 * 25 rows (1,250 rows)
```

**Data:**

```text
- `orders`: order_id (int), product_id (int), category_id (int), order_amount (float)
- `products`: id (int), product_name (string), category_id (int)
- `categories`: id (int), category_name (string)
```

**Flow:**

1. Fan-out
2. Inflation
3. Initial Query
4. Join Orders with Products
5. JOIN Orders with Categories
6. SUM Order Amounts (125,000rows)
7. Pre-aggregation Approach
8. Aggregate Orders by Product
9. Join Aggregated Data withProducts & Categories
10. Calculate Final Result (1,250rows)

