---
type: "practice"
difficulty: "Deep"
---

# When joining three tables Orders Customers and Products what order does the join get evaluated in logically?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Join Practice Problems](<../../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Join_Practice_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how a query with joins between Orders, Customers, and Products gets evaluated, we need to break down the steps of the SQL evaluation process from first principles. The key is to recognize that the order in which tables are joined can affect performance, but logically, joins happen one by one starting from the innermost join.

#### Triage

The question asks about the logical evaluation order of a query joining three tables: Orders, Customers, and Products. We need to determine the sequence in which these tables are joined. The core idea is that the database engine processes each join step by step, evaluating one pair of tables at a time.

#### First-Principles Solution
Inner Join: SQL evaluates joins from left to right.
Step-by-Step Evaluation:
First, it performs an inner join between Orders and Customers.
Then, it uses the result of this join as one side of a new join with Products.
Detailed Explanation
Initial Setup: Assume you have three tables: Orders, Customers, and Products. Each table has its own set of columns.
First Join (Orders & Customers):
The database engine starts by joining the Orders and Customers tables using their common keys (e.g., customer_id).
This produces a new temporary result set that combines data from both tables based on matching rows in the two tables.
Second Join (Result of First Join & Products):
The engine then takes this temporary result set and joins it with the Products table using another common key, such as product_id.

#### Example
Query

Here's a simple example query to illustrate:

SQL
SELECT *
FROM Orders
JOIN Customers ON Orders.customer_id = Customers.customer_id
JOIN Products ON Orders.product_id = Products.product_id;
Orders & Customers Join:
The engine processes the Orders table first.
It then looks at the Customers table and performs an inner join based on matching customer_id.
Result of First Join (Temp Table A)
This step produces a temporary result set with columns from both Orders and Customers.
Second Join (Temp Table A & Products):
The engine then takes the temporary result set and joins it with the Products table based on matching product_id.

#### Performance Considerations
Join Order: While this is the logical order, performance can vary depending on the join order due to indexing and data distribution.
Indexing: Proper indexing on key columns used in joins (e.g., customer_id, product_id) can significantly improve performance.

#### Closing Summary
+ Common Mistakes

The order of evaluation for a query with multiple joins is from left to right, starting with the first inner join. The common mistake is assuming that the database optimizes the join process differently; however, it follows this logical sequence unless explicitly optimized through a different query structure or indexing strategy.

Always consider the performance implications and use appropriate indexes to optimize your queries.

