---
type: "practice"
difficulty: "Intermediate"
---

# Write a SQL query to count orders per customer city joining fact and dimension tables.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Fact and Dimension Keys](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Fact_and_Dimension_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To answer this question, we need to write an SQL query that counts the number of orders placed by customers grouped by their city. This involves joining the orders (fact) table with the customers (dimension) table on a common key like customer_id. The core idea is to use an aggregate function, such as COUNT, in conjunction with the GROUP BY clause.

#### Triage

The question asks for a SQL query that counts orders per customer city. This means we need to join two tables: one containing order information (fact table) and another containing customer details (dimension table). We will use an aggregate function to count the number of orders, grouped by customer_city.

#### First-Principles Solution

Assuming the following schema:

The orders table has columns order_id, customer_id, and others.
The customers table has columns customer_id, first_name, last_name, and customer_city.

The SQL query will look like this:

SQL
SELECT
    c.customer_city,
    COUNT(o.order_id) AS order_count
FROM
    orders o
JOIN
    customers c ON o.customer_id = c.customer_id
GROUP BY
    c.customer_city;

#### Closing Summary
and Common Mistakes/Follow-Ups

This query correctly joins the orders table with the customers table, counts the number of orders for each city, and groups them accordingly. Key points to remember include using appropriate join conditions and ensuring that the schema matches the assumptions made.

#### Common mistakes
might include:

Forgetting to use an aggregate function like COUNT.
Using incorrect column names or mismatched keys.
Not including a GROUP BY clause which is necessary for aggregating data by city.

For further refinement, consider:

Adding additional columns from the customers table if needed.
Filtering orders based on specific criteria (e.g., date range) if required.
Optimizing the query performance if dealing with large datasets, possibly using indexes or partitioning strategies.

**Code:**

```sql
SELECT
    c.customer_city,
    COUNT(o.order_id) AS order_count
FROM
    orders o
JOIN
    customers c ON o.customer_id = c.customer_id
GROUP BY
    c.customer_city;
```

