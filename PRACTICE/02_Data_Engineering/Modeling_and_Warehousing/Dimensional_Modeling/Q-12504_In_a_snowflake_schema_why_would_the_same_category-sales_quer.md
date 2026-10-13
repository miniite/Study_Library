---
type: "practice"
difficulty: "Intermediate"
---

# In a snowflake schema, why would the same category-sales query need extra joins?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Star Schemas and Dimension Keys](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Star_Schemas_and_Dimension_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

A key reason for needing additional joins in a snowflake schema is due to its denormalized structure, where dimension tables are further normalized into separate smaller tables. This results in more complex queries that require traversing these relationships.

#### Triaging the Question

The question revolves around understanding why a query involving category and sales data might need extra joins when using a snowflake schema compared to a star schema. The interviewer is probing your knowledge of how data models are structured, the trade-offs between different schemas, and your ability to reason through the implications of each structure on query performance.

#### First-Principles Solution
Star Schema vs Snowflake Schema

In a star schema, all fact tables directly reference dimension tables. For example, if you have sales data, it would link directly to customer, time, product dimensions without any intermediary layers. This results in simpler queries and faster execution times but might lead to larger join operations.

In contrast, the snowflake schema normalizes the dimension tables, creating a hierarchy where each level of a dimension is represented as a separate table. For instance, instead of having a single product table, you could have category, subcategory, and product. This reduces data redundancy but increases query complexity.

Query Complexity in Snowflake Schema

#### Cons
ider a scenario where you need to fetch the total sales for each category. In a star schema, this would involve joining the sales fact table directly with the product dimension table (or possibly other dimensions like time). However, in a snowflake schema, you would first join the sales fact table with the product table, then traverse the hierarchy of categories and subcategories.

#### Example
Query

Suppose we have the following tables:

Sales: Contains sales data linked to product IDs.
Product: Contains basic product information (product ID, category ID).
Category: Hierarchy: Category -> Subcategory. Each row has a category_id and optionally a subcategory_id.

The query in a star schema might look like this:

SQL
SELECT SUM(sales.amount) AS total_sales
FROM Sales s
JOIN Product p ON s.product_id = p.product_id;

In a snowflake schema, the same query would need to traverse the category hierarchy:

SQL
SELECT SUM(s.sales_amount) AS total_sales
FROM Sales s
JOIN Product p ON s.product_id = p.product_id
LEFT JOIN Category c ON p.category_id = c.category_id
LEFT JOIN Subcategory sc ON p.subcategory_id = sc.subcategory_id;

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, the extra joins in a snowflake schema are necessary to navigate the hierarchical structure of dimension tables. This results in more complex queries but can improve data integrity and scalability.

#### Common mistakes
include overlooking the increased query complexity or not understanding why the snowflake schema is used despite its potential drawbacks. Follow-ups might include discussing performance optimization techniques, such as materialized views or partitioning strategies, to mitigate the impact of additional joins.

Common Edge Cases
Performance Impact: Snowflake schemas can be slower due to more complex queries.
Data Integrity: Better handling of hierarchical data but requires careful schema design.

**Comparison:**

```
JOIN
Sales
Product
Category
Subcategory
```

**Code:**

```sql
-- Example query in star schema
SELECT SUM(sales.amount) AS total_sales
FROM Sales s
JOIN Product p ON s.product_id = p.product_id;

-- Example query in snowflake schema
SELECT SUM(s.sales_amount) AS total_sales
FROM Sales s
JOIN Product p ON s.product_id = p.product_id
LEFT JOIN Category c ON p.category_id = c.category_id
LEFT JOIN Subcategory sc ON p.subcategory_id = sc.subcategory_id;
```

**Example input / output:**

```

Input:

Sales table with product sales data.
Product table with category and subcategory IDs.

Output:

Total sales for each category (sum of sales amount).

Example Request:

SQL
Copy
SELECT SUM(sales.amount) AS total_sales
FROM Sales s
JOIN Product p ON s.product_id = p.product_id
LEFT JOIN Category c ON p.category_id = c.category_id
LEFT JOIN Subcategory sc ON p.subcategory_id = sc.subcategory_id;

Response:

The query processes all sales data and aggregates them based on the category hierarchy.
```

**Data:**

```json
{
  "Sales": [
    {"product_id": 1, "sales_amount": 50},
    {"product_id": 2, "sales_amount": 75}
  ],
  "Product": [
    {"product_id": 1, "category_id": 1},
    {"product_id": 2, "category_id": 1, "subcategory_id": 3}
  ],
  "Category": [{"category_id": 1, "name": "Electronics"}],
  "Subcategory": [{"subcategory_id": 3, "name": "Cameras"}]
}
```

**Flow:**

1. Database
2. User
3. Database
4. User
5. SELECT SUM(sales.amount) AS total_sales FROM Sales s JOIN Product p ON s.product_id = p.product_id LEFT JOIN Category c ON p.category_id = c.category_id LEFT JOIN Subcategory sc ON p.subcategory_id = sc.subcategory_id
6. Total sales for each category

