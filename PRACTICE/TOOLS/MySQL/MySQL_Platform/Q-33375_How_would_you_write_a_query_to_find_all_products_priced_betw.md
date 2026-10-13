---
type: "practice"
difficulty: "Basic"
---

# How would you write a query to find all products priced between 100 and 500?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Basic Query Practice](<../../../../TOOLS/MySQL/MySQL_Platform/Basic_Query_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To write a SQL query in MySQL that finds all products with prices between 100 and 500, you need to understand the basic syntax of SELECT statements and how to use range conditions.

SQL
-- SQL Query to find products priced between 100 and 500
SELECT *
FROM products
WHERE price >= 100 AND price <= 500;

This query selects all columns from the products table where the price is greater than or equal to 100 and less than or equal to 500.

The query is correct and complete.

**Code:**

```sql
-- SQL Query to find products priced between 100 and 500
SELECT *
FROM products
WHERE price >= 100 AND price <= 500;

This query selects all columns from the products table where the price is greater than or equal to 100 and less than or equal to 500.
```

