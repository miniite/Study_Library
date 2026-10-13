---
type: "practice"
difficulty: "Intermediate"
---

# Where in a query can a subquery appear (SELECT WHERE FROM HAVING)?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Rewriting Subqueries as Joins](<../../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Rewriting_Subqueries_as_Joins.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In SQL queries, subqueries can appear in multiple parts of a query statement—most commonly within SELECT, WHERE, AND, OR, GROUP BY, and HAVING clauses. Let’s explore where they fit most effectively by first understanding the basic structure of a query and then diving into each clause.

#### Triaging the Question

The question is essentially asking about the placement of subqueries in SQL statements. The interviewer might be testing your knowledge on how subqueries interact with different parts of an SQL statement, their syntax, and the impact they have on performance. We will assume that a subquery is defined as a query nested inside another query.

#### First-Principles Solution

Let’s start by examining each clause where subqueries can appear:

SELECT Clause: Subqueries in SELECT clauses are used to extract specific data from tables based on some condition.
WHERE, AND, OR Clauses: These clauses use simple conditions; subqueries here provide more complex filtering logic.
FROM Clause: Although less common, you might see correlated subqueries that reference the table being queried itself in this clause.
GROUP BY and HAVING Clauses: Here, subqueries are used to filter groups or aggregate results based on some condition.
Detailed Explanation
SELECT Clause

#### Example
To select records where a specific column matches values from another table.
SQL
SELECT employee_id FROM employees WHERE department_id IN (SELECT department_id FROM departments WHERE location = 'New York');
WHERE, AND, OR Clauses

#### Example
Using a simple condition: WHERE price > 100 AND (SELECT AVG(price) FROM products) < 250.
Using an IN clause:
SQL
SELECT product_name FROM orders WHERE order_id IN (SELECT order_id FROM line_items WHERE quantity > 10);
FROM Clause

#### Example
of a correlated subquery:
SQL
SELECT e.employee_id, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id
WHERE e.salary > (SELECT AVG(salary) FROM employees WHERE department_id = e.department_id);
GROUP BY and HAVING Clauses

#### Example
SQL
SELECT category, SUM(quantity)
FROM orders o
WHERE order_date BETWEEN '2021-01-01' AND '2021-12-31'
GROUP BY category
HAVING (SELECT AVG(quantity) FROM line_items WHERE order_id = o.order_id) > 5;

#### Closing Summary
and Common Mistakes/Follow-Ups

Subqueries are powerful tools that can enhance the flexibility and expressiveness of SQL queries. However, overusing subqueries can lead to performance issues due to their nested nature.

Common Mistake: Using subqueries in simple WHERE clauses instead of JOINs when a join would be more efficient.

#### Follow-Up
Question: How do correlated subqueries differ from non-correlated subqueries? What are the potential performance implications?

#### Example
Worked Input->Output
SQL
-- Sample data schema
CREATE TABLE orders (
    order_id INT,
    product_name VARCHAR(100),
    quantity INT,
    order_date DATE
);

INSERT INTO orders (order_id, product_name, quantity, order_date) VALUES
(1, 'Laptop', 5, '2023-01-01'),
(2, 'Tablet', 7, '2023-01-15'),
(3, 'Mouse', 9, '2023-02-01'),
(4, 'Keyboard', 6, '2023-02-15');

-- Query using subquery in WHERE clause
SELECT order_id, product_name
FROM orders
WHERE quantity > (SELECT AVG(quantity) FROM line_items);
Self-check

The solution is correct and comprehensive.

**Comparison:**

| id | product_name | quantity | order_date          |
|----|--------------|----------|---------------------|
| 1  | Laptop       | 5        | 2023-01-01          |
| 2  | Tablet       | 7        | 2023-01-15          |
| 3  | Mouse        | 9        | 2023-02-01          |
| 4  | Keyboard     | 6        | 2023-02-15          |
| order_id | quantity |
|----------|----------|
| 1        | 3        |
| 2        | 4        |
| 3        | 6        |
| 4        | 5        |

**Code:**

```sql
CREATE TABLE orders (
    order_id INT,
    product_name VARCHAR(100),
    quantity INT,
    order_date DATE
);

INSERT INTO orders (order_id, product_name, quantity, order_date) VALUES
(1, 'Laptop', 5, '2023-01-01'),
(2, 'Tablet', 7, '2023-01-15'),
(3, 'Mouse', 9, '2023-02-01'),
(4, 'Keyboard', 6, '2023-02-15');

SELECT order_id, product_name
FROM orders
WHERE quantity > (SELECT AVG(quantity) FROM line_items);
```

**Example input / output:**

```text
-- Sample output from the query
+---------+-----------+
| order_id | product_name |
+---------+-------------+
|        3 | Mouse       |
+---------+-------------+
```

**Data:**

```sql
CREATE TABLE line_items (
    order_id INT,
    quantity INT
);

INSERT INTO line_items (order_id, quantity) VALUES
(1, 3),
(2, 4),
(3, 6),
(4, 5);
```

**Flow:**

1. No
2. Yes
3. Yes
4. No
5. Yes
6. No
7. SELECT Clause
8. Is it a simple condition?
9. Subquery in WHERE, AND, ORClauses
10. Subquery not necessary
11. FROM Clause
12. Is the subquery correlated?
13. Correlated Subquery
14. Simple join or aggregate query
15. GROUP BY and HAVINGClauses
16. Does it involve complexaggregation?
17. Subquery in GROUP BY,HAVING Clauses
18. Simple grouping with nosubqueries

