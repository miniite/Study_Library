---
type: "practice"
difficulty: "Intermediate"
---

# What is a subquery and how does a correlated subquery differ from a non-correlated one?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ MySQL Optimizer Behavior](<../../../../TOOLS/MySQL/MySQL_Platform/MySQL_Optimizer_Behavior.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Subqueries are fundamental in SQL to include query results within another query, allowing complex logic and data filtering. A non-correlated subquery is evaluated independently of the outer query, while a correlated subquery's execution is tied to each row processed by the outer query. This distinction significantly impacts performance and complexity.

In essence:

Non-correlated subqueries are evaluated once for the entire outer query.
Correlated subqueries evaluate repeatedly for each row in the outer query, which can be resource-intensive but more flexible.

Let's dive into these concepts with a detailed explanation and examples.

#### Triage
Understanding Subqueries
What is a Subquery?

A subquery is an SQL query nested within another SQL statement. It returns data that are then used as part of the main query’s conditions or operations.

Correlated vs Non-correlated Subqueries
Non-correlated subqueries do not depend on any columns in the outer query and can be evaluated independently.
Correlated subqueries use values from the outer query, which makes them dependent on each row being processed by the outer query.
First Principles: Non-correlated Subqueries

#### Example
1: Simple Filter

#### Cons
ider a users table with an orders table. To find users who have ordered more than one product:

SQL
SELECT * FROM users WHERE user_id IN (SELECT user_id FROM orders GROUP BY user_id HAVING COUNT(*) > 1);

Here, the subquery (SELECT user_id FROM orders GROUP BY user_id HAVING COUNT(*) > 1) is evaluated independently and does not reference any columns from the users table. It returns a list of users who have placed more than one order.

Performance Consideration

Non-correlated subqueries can be optimized well by the database engine, as they do not depend on row-level data in the outer query. They are often faster but may still involve multiple scans if not indexed properly.

First Principles: Correlated Subqueries

#### Example
2: Conditional Filtering

To find users who have ordered products that cost more than $10:

SQL
SELECT * FROM users WHERE user_id IN (SELECT user_id FROM orders WHERE order_amount > 10);

In this example, the subquery (SELECT user_id FROM orders WHERE order_amount > 10) uses a column from the orders table and thus is correlated to each row in the users table. The database must re-evaluate the subquery for each user_id.

Performance Consideration

Correlated subqueries can be less efficient because they evaluate every time a new row in the outer query is considered, leading to multiple scans of the inner query.

Practical Implementation

Let's compare these concepts with an example:

SQL
-- Non-correlated subquery: users with more than one order
SELECT * FROM users WHERE user_id IN (SELECT user_id FROM orders GROUP BY user_id HAVING COUNT(*) > 1);

-- Correlated subquery: users who ordered expensive items
SELECT * FROM users WHERE EXISTS (
    SELECT 1 FROM orders WHERE user_id = users.user_id AND order_amount > 10
);

In the correlated subquery, EXISTS is used to check if there exists an order with a value greater than $10 for each user. This example is more complex but can be more efficient in certain scenarios.

#### Closing Summary
and Common Mistakes

Correlated subqueries are powerful but can lead to performance issues due to their row-level dependency. Non-correlated subqueries, while simpler, might not offer the same flexibility. Understanding these differences helps in writing effective SQL queries that balance between performance and complexity.

#### Common Mistakes

Not considering indexing strategies.
Overusing correlated subqueries for operations that could be handled by joins or non-correlated subqueries.
Misunderstanding the impact of correlated subquery execution on overall query performance.

By carefully choosing when to use each type, you can optimize your SQL queries and maintain efficient database operations.

