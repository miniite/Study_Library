---
type: "practice"
difficulty: "Intermediate"
---

# How do you retrieve rows from one table that have no matching row in another, using a LEFT JOIN?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Join Types](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Join_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To retrieve rows from one table that have no matching row in another using a LEFT JOIN, we need to understand how SQL joins work and their different types. In this case, we are specifically looking at the LEFT JOIN concept.

#### Triage

The question is asking us to identify records in one table (let's call it Table A) that do not have corresponding entries in another table (Table B). This involves using a LEFT JOIN where we match rows from both tables based on a common key, and return all rows from the left (Table A), even if there is no match in the right table (Table B).

#### Assumptions
Both tables share at least one common column used as the join condition.
The query should be optimized for performance, especially with large datasets.

#### First-principles Solution

A LEFT JOIN between two tables can achieve this. Here’s how you can structure it:

Identify Common Columns: Determine which columns are used to match records in both tables.

#### Cons
truct the SQL Query: Use a LEFT JOIN clause, ensuring that the query returns all rows from Table A and any matching rows from Table B.

#### Example

Suppose we have two tables: Customers (Table A) and Orders (Table B). We want to find customers who do not have any orders. The SQL query would look like this:

SQL
SELECT Customers.customer_id, Customers.name, Orders.order_id
FROM Customers
LEFT JOIN Orders ON Customers.customer_id = Orders.customer_id
WHERE Orders.order_id IS NULL;

#### Closing Summary
and Common Mistakes/Follow-Ups:

In the above example, we selected all columns from Customers and included the order_id column from Orders. The WHERE clause filters out rows where there is no match in Orders, thus returning only customers without any orders.

#### Common mistakes
include using an incorrect join type (e.g., INNER JOIN instead of LEFT JOIN) or forgetting to check for null values. Additionally, not clearly identifying the common key between the tables can lead to errors.

If asked follow-up questions, you might be probed about performance optimizations, handling large datasets, or dealing with more complex scenarios involving multiple joins or subqueries.

**Code:**

```sql
SELECT Customers.customer_id, Customers.name, Orders.order_id
FROM Customers
LEFT JOIN Orders ON Customers.customer_id = Orders.customer_id
WHERE Orders.order_id IS NULL;
```

