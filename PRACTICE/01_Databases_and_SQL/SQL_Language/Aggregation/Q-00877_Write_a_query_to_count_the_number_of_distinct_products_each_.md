---
type: "practice"
difficulty: "Intermediate"
---

# Write a query to count the number of distinct products each customer has ever bought.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Counting with Joins](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/Counting_with_Joins.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, you need to write a SQL query that counts the number of distinct products purchased by each customer. The key is to use an aggregate function and group by clause effectively. Let's break it down step-by-step.

#### Triaging the Question

The question asks us to count unique products per customer based on their purchase history. We are likely dealing with a purchases table that contains at least two columns: customer_id (the ID of the customer) and product_id (the ID of the product). The goal is to determine how many distinct products each customer has ever bought.

#### Assumptions

The purchases table has a schema like this:

SQL
CREATE TABLE purchases (
    purchase_id INT PRIMARY KEY,
    customer_id INT NOT NULL,
    product_id INT NOT NULL,
    purchase_date DATE
);

We want to return a list of customers along with the number of distinct products they have bought.

#### First Principles Solution
Identify the relevant columns: customer_id and product_id.
Determine the aggregation needed: Use the COUNT() function in combination with DISTINCT to count unique product IDs for each customer.
Group by customer ID: To get a distinct count per customer, we need to group the results by customer_id.
SQL Query

The query will look something like this:

SQL
SELECT
    customer_id,
    COUNT(DISTINCT product_id) AS num_distinct_products
FROM
    purchases
GROUP BY
    customer_id;

#### Closing Summary
and Common Mistakes/Follow-Ups

This query correctly counts the number of distinct products each customer has ever bought. However, there are a few common mistakes to watch out for:

Not using DISTINCT: If you omit DISTINCT, you might get an incorrect count because duplicate product IDs would be counted multiple times.
Incorrect grouping column: Always ensure that the columns in your GROUP BY clause align with what you want to aggregate over.

If the interviewer asks follow-up questions, they might probe further into:

How to handle customers who haven't made any purchases yet (e.g., excluding them or including them as having zero distinct products).

#### Performance considerations
if the table is very large.
Handling null values in product_id columns.
Joining with additional tables like a customers table to get customer names alongside counts.

**Code:**

```sql
SELECT
    customer_id,
    COUNT(DISTINCT product_id) AS num_distinct_products
FROM
    purchases
GROUP BY
    customer_id;
```

