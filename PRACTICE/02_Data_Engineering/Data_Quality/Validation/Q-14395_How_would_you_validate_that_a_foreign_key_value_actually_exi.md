---
type: "practice"
difficulty: "Intermediate"
---

# How would you validate that a foreign key value actually exists in the referenced dimension table?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Uniqueness Checks](<../../../../02_Data_Engineering/Data_Quality/Validation/Uniqueness_Checks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To validate that a foreign key (FK) value actually exists in the referenced dimension table, we need to ensure data consistency and integrity across our database tables. This validation step is crucial to avoid orphaned records or incorrect references. Here’s how you can approach this problem:

Understanding the Question: The core question here is about ensuring that every FK in a table points to an existing row in its corresponding dimension table. We need to check for missing references and ensure that all values are correctly linked.

Key Assumptions:

Both tables have primary keys (PK) and foreign keys.
The dimension table contains valid data, i.e., no NULL or invalid entries in the PK column.
You will be working with SQL databases like PostgreSQL, MySQL, etc.

#### First-Principles Solution

Step 1: Identify the FK columns in the tables that need validation.
Step 2: Query the referenced dimension table to get all valid values for these columns.
Step 3: Join or use a subquery to check if each FK value from the main table exists in the dimension table.

Full Solution:

SQL
-- Assuming we have two tables: `orders` and `customers`
-- orders(customer_id, order_date)
-- customers(id, name)

SELECT o.customer_id, c.id AS customer_id_in_dim_table
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.id
WHERE c.id IS NULL;

This SQL query will return all customer_id values in the orders table that do not exist in the customers table. If no rows are returned, it means every FK value is valid.

#### Closing Summary
By executing this query, we can identify any invalid foreign key references and take corrective action, such as cleaning up the data or updating the records to correct the reference.

#### Common Mistakes/Follow-Ups

Forgetting to check for NULL values in the FK column.
Not considering other constraints like uniqueness on the FK column if applicable.
Overlooking potential performance issues with large datasets and ensuring proper indexing.
Assuming that all tables have a well-defined primary key structure.

