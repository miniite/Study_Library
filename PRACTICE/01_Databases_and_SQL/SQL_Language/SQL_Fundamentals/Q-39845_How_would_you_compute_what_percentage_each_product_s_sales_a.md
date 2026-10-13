---
type: "practice"
difficulty: "Intermediate"
---

# How would you compute what percentage each product's sales are of the total sales in one query?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Aggregation Query Practice](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Aggregation_Query_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To calculate the percentage of each product’s sales as a proportion of the total sales, we need to follow a few steps and ensure our SQL query is structured correctly. This involves summing up the total sales across all products and then dividing each individual product's sales by this total sum, before multiplying by 100 to get the percentage.

**Comparison:**

| product_id | product_name    | sales_amount | sales_percentage   |
| ---       | ---             | ---          | ---                |
| 101       | Laptop          | 5000         | 50.00              |
| 202       | Smartphone      | 3000         | 30.00              |
| 303       | Headphones      | 2000         | 20.00              |

**Code:**

```sql
-- Step-by-step SQL Query

WITH TotalSales AS (
    SELECT SUM(sales_amount) as total_sales FROM sales_table
),
ProductSales AS (
    SELECT
        product_id,
        product_name,
        sales_amount,
        (sales_amount / (SELECT total_sales FROM TotalSales)) * 100 as sales_percentage
    FROM sales_table
)

-- To get the final result with all products and their respective percentages
SELECT
    ps.product_id,
    ps.product_name,
    ps.sales_amount,
    ROUND(ps.sales_percentage, 2) AS sales_percentage
FROM ProductSales ps;
```

**Example input / output:**

```text
-- Sample Input
WITH TotalSales AS (
    SELECT SUM(sales_amount) as total_sales FROM sales_table
),
ProductSales AS (
    SELECT
        product_id,
        product_name,
        sales_amount,
        (sales_amount / (SELECT total_sales FROM TotalSales)) * 100 as sales_percentage
    FROM sales_table
)

-- Final Query Output
SELECT
    ps.product_id,
    ps.product_name,
    ps.sales_amount,
    ROUND(ps.sales_percentage, 2) AS sales_percentage
FROM ProductSales ps;

-- Expected Output (assuming total_sales = 10000)
+---------+-----------------+-------------+--------------------+
| product_id | product_name    | sales_amount | sales_percentage   |
+---------+-----------------+-------------+--------------------+
| 101     | Laptop          | 5000        | 50.00              |
+---------+-----------------+-------------+--------------------+
| 202     | Smartphone      | 3000        | 30.00              |
+---------+-----------------+-------------+--------------------+
| 303     | Headphones      | 2000        | 20.00              |
+---------+-----------------+-------------+--------------------+
```

**Data:**

```json
{
    "sales_table": [
        {"product_id": 101, "product_name": "Laptop", "sales_amount": 5000},
        {"product_id": 202, "product_name": "Smartphone", "sales_amount": 3000},
        {"product_id": 303, "product_name": "Headphones", "sales_amount": 2000}
    ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is there a sales table?
5. Calculate Total Sales
6. Calculate Product-wise Sales
7. Determine the total salesamount from the product-wisesum
8. Compute Percentage for EachProduct
9. Output Final Result
10. End

