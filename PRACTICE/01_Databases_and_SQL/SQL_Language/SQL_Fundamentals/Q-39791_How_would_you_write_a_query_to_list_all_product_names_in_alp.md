---
type: "practice"
difficulty: "Basic"
---

# How would you write a query to list all product names in alphabetical order ignoring case?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Data Validation Queries](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Data_Validation_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to retrieve all product names from a table and sort them in alphabetical order while ignoring their case. This involves using SQL's SELECT statement with an appropriate ORDER BY clause.

#### Triage
Question: How do you write a query that lists all product names in alphabetical order without considering the case sensitivity?
Key Idea: Use SQL's LOWER() function to ensure case-insensitive sorting.

#### Assumptions
The table name is products and there is a column named product_name.

#### Solution

First, let’s start by writing the basic query to select all product names:

SQL
SELECT product_name FROM products;

Next, we need to sort these names in alphabetical order. We can use the ORDER BY clause for this purpose. However, SQL's default sorting is case-sensitive. To make it case-insensitive, we will wrap the column name with the LOWER() function:

SQL
SELECT product_name FROM products ORDER BY LOWER(product_name);

#### Closing Summary

The query above lists all product names in alphabetical order ignoring their case sensitivity.

#### Common Mistakes/Follow-Ups
Incorrect Use of Functions: Not using LOWER(), which can lead to case-sensitive sorting.
Misplaced Function Usage: Using the function inside the select list instead of ORDER BY, e.g., SELECT LOWER(product_name) FROM products ORDER BY product_name; would still be case-sensitive.
Depth and Language Matched for Freshers:

For beginners, it's crucial to understand the basic concepts well before moving on. The example given here is straightforward but essential. Understanding how SQL functions like LOWER() work can significantly improve your ability to manipulate data in various ways.

**Code:**

```sql
SELECT product_name FROM products ORDER BY LOWER(product_name);
```

