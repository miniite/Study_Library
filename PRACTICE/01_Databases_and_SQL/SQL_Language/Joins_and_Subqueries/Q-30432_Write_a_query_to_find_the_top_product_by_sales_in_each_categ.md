---
type: "practice"
difficulty: "Deep"
---

# Write a query to find the top product by sales in each category using a subquery.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Subquery Practice Problems](<../../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Subquery_Practice_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To find the top product by sales in each category, we need to write a SQL query that involves both joins and subqueries. The main idea is to use a subquery to identify the highest sales amount within each category and then join this with the products table to get the details of those top-selling products.

#### Triaging the Question

The question asks us to find the top product by sales in each category. This involves:

Grouping: Grouping the data by category.
Aggregation: Using an aggregate function (like SUM or MAX) to calculate the total sales for each group.
Ordering and Limitation: Ordering the results within each category based on sales and limiting to one record per category.

#### Assumptions
We have a products table with columns such as product_id, category_id, name, and sales.
The sales column represents the total sales of the product.
Products within the same category can have different sales amounts, and we need to identify the top-selling one in each category.
Step-by-Step Solution
Subquery: First, create a subquery that calculates the maximum sales for each category.
Join: Join this result with the products table to get the product details of the top-selling products.

Here is the SQL query:

SQL
SELECT p.*
FROM products p
JOIN (
    SELECT category_id, MAX(sales) AS max_sales
    FROM products
    GROUP BY category_id
) ps ON p.category_id = ps.category_id AND p.sales = ps.max_sales;
Explanation
The subquery first groups the products table by category_id and calculates the maximum sales for each category. This is stored in a temporary result set with columns category_id and max_sales.
The main query then joins this subquery back to the products table on both category_id and sales. Only rows where the sales amount matches the maximum sales within the category are returned, effectively giving us the top-selling product for each category.

#### Closing Summary

This solution ensures that we find the top-selling product in each category by leveraging a subquery to calculate the necessary aggregate values. The query is efficient as it minimizes the number of operations and leverages database indexing on category_id and sales columns if they are properly indexed.

#### Common Mistakes/Follow-Ups
Incorrect Join Condition: Make sure that both category_id and sales conditions in the join clause match the subquery's output.
Handling Ties: If multiple products can have the same maximum sales amount within a category, this query will return all such products. To handle ties more explicitly, you might need to use additional logic or window functions depending on your requirements.

**Comparison:**

| product_id | category_id | name      | sales |
|------------|-------------|-----------|-------|
| 101        | 2           | Product A | 500   |
| 102        | 2           | Product B | 450   |
| 103        | 3           | Product C | 600   |
| 104        | 3           | Product D | 600   |

**Code:**

```sql
SELECT p.*
FROM products p
JOIN (
    SELECT category_id, MAX(sales) AS max_sales
    FROM products
    GROUP BY category_id
) ps ON p.category_id = ps.category_id AND p.sales = ps.max_sales;
```

