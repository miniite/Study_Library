---
type: "practice"
difficulty: "Basic"
---

# Write a query to find the total inventory value across all warehouses.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Counting with Joins](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/Counting_with_Joins.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To write a SQL query that finds the total inventory value across all warehouses, we need to consider how our data is structured and what aggregate functions are available in SQL. The key concept here is using an SUM function to add up values from multiple rows.

#### Triaging the Question

The question asks for a query to find the total inventory value across all warehouses. This means we need to sum the inventory_value (or whatever column name represents the monetary value of each item) across all records in our table, grouped by warehouses if applicable.

#### Assumptions
and First Principles
We have a table named warehouses with columns such as warehouse_id, product_name, quantity, and unit_price.
The total inventory value is calculated as quantity * unit_price.
Step-by-Step Solution
Identify the Table: Let’s assume our table is called inventory.
Aggregate Function: Use the SUM function to aggregate the total value.
Join Components: Ensure we correctly multiply quantity by unit price.

Here is a step-by-step breakdown of constructing this query:

SQL
SELECT
    SUM(quantity * unit_price) AS total_inventory_value
FROM
    inventory;

#### Closing Summary
and Common Mistakes

This simple query will give you the total inventory value across all warehouses. However, there are common pitfalls to watch out for:

Ensure that quantity and unit_price columns exist in your table.
Double-check column names as they might differ based on schema design.
Code Block
SQL
SELECT
    SUM(quantity * unit_price) AS total_inventory_value
FROM
    inventory;

**Code:**

```sql
SELECT
    SUM(quantity * unit_price) AS total_inventory_value
FROM
    inventory;
```

