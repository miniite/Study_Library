---
type: "practice"
difficulty: "Deep"
---

# Explain how a large IN-subquery, a JOIN, and an EXISTS can produce three different plans for 'orders whose customer is premium', and how inner-set row estimates decide which wins.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Join Types](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Join_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how IN-subqueries, JOIN, and EXISTS can produce three distinct query execution plans when filtering orders by a customer's premium status, we need to delve into the inner workings of SQL optimizers and their heuristics for row estimation. Let’s break down each approach: using an IN subquery, performing a JOIN, or leveraging an EXISTS. We’ll also explore how row estimates influence which plan is chosen by the optimizer.

#### Triaging the Question

The question asks us to compare three SQL query approaches—IN-subquery, JOIN, and EXISTS—for filtering orders based on a customer's premium status. The key aspect here is understanding the differences in execution plans these constructs can produce due to varying row estimates.

#### First Principles Solution
1. IN Subquery

An IN subquery involves checking if an order’s customer ID is present in a set of values returned by another query (the subquery). The optimizer typically uses a nested loop join for this, where the outer query (orders) iterates over each row and checks membership in the inner query result.

#### Example
Query:

SQL
SELECT * FROM orders WHERE customer_id IN (SELECT id FROM customers WHERE is_premium = true);
2. JOIN

A JOIN operation combines rows from two tables based on a related column between them. In this case, we can join the orders and customers tables directly.

#### Example
Query:

SQL
SELECT * FROM orders o JOIN customers c ON o.customer_id = c.id WHERE c.is_premium = true;
3. EXISTS Subquery

The EXISTS subquery checks if a query returns any rows, without needing to fetch all the data. It short-circuits and stops as soon as it finds at least one matching row.

#### Example
Query:

SQL
SELECT * FROM orders WHERE EXISTS (SELECT 1 FROM customers WHERE customer_id = o.customer_id AND is_premium = true);
Row Estimates and Plan Selection

The SQL optimizer uses estimated numbers of rows (row_estimate) to decide which plan is most efficient. Here’s how each query could be evaluated:

IN Subquery
Row Estimate: The optimizer estimates the number of customer IDs that match is_premium = true. If there are many such customers, this can lead to a large number of nested loop operations.
Plan: Nested Loop Join (NLJ) is typically chosen due to its simplicity.
JOIN
Row Estimate: The optimizer estimates the join cardinality between orders and customers. This involves counting how many orders each customer has and estimating the overlap based on their premium status.
Plan: Nested Loop Join or Hash Join might be used, depending on the estimated sizes of both tables.
EXISTS Subquery
Row Estimate: The optimizer checks if there are any customers who are premium. If only a few or none match, this can significantly reduce the number of rows in the orders table.
Plan: Index Seek or NLJ might be used; the plan is often optimized to stop early once it finds the first matching row.

#### Common Mistakes
and Follow-ups
Incorrect Row Estimates: If the optimizer’s estimates are off, a suboptimal plan may be chosen. For example, if it overestimates the number of premium customers, an IN subquery might appear more efficient than it really is.
Index Usage: Ensuring that appropriate indexes are in place can significantly impact performance. An index on customers(is_premium) could drastically reduce the cost of both EXISTS and JOIN plans.
Performance Tuning: Understanding how to manually influence query optimization, such as using hints or rewriting queries, is crucial for complex systems.

#### Closing Summary

Understanding the differences in execution plans between IN, JOIN, and EXISTS subqueries hinges on row estimates and cardinality considerations. The optimizer selects a plan based on these factors, which can vary greatly depending on the actual data distribution.

Self-check

This answer covers all aspects of the question, including triage, key concepts, and common pitfalls in query optimization.

**Comparison:**

| order_id | customer_id | product_name |
|----------|-------------|--------------|
| 101      | 1           | Laptop       |
| 102      | 2           | Mouse        |
| 103      | 3           | Keyboard     |
| id | name      | is_premium |
|----|-----------|------------|
| 1  | Alice     | TRUE       |
| 2  | Bob       | FALSE      |
| 3  | Charlie   | TRUE       |

**Code:**

```sql
-- IN subquery example
SELECT * FROM orders WHERE customer_id IN (SELECT id FROM customers WHERE is_premium = true);

-- JOIN example
SELECT * FROM orders o JOIN customers c ON o.customer_id = c.id WHERE c.is_premium = true;

-- EXISTS subquery example
SELECT * FROM orders WHERE EXISTS (SELECT 1 FROM customers WHERE customer_id = o.customer_id AND is_premium = true);
```

**Example input / output:**

```text
-- IN subquery
SELECT * FROM orders WHERE customer_id IN (SELECT id FROM customers WHERE is_premium = true);

-- JOIN example
SELECT * FROM orders o JOIN customers c ON o.customer_id = c.id WHERE c.is_premium = true;

-- EXISTS subquery
SELECT * FROM orders WHERE EXISTS (SELECT 1 FROM customers WHERE customer_id = o.customer_id AND is_premium = true);
```

**Data:**

```sql
CREATE TABLE customers (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    is_premium BOOLEAN
);

INSERT INTO customers VALUES
(1, 'Alice', TRUE),
(2, 'Bob', FALSE),
(3, 'Charlie', TRUE);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    product_name VARCHAR(255)
);

INSERT INTO orders VALUES
(101, 1, 'Laptop'),
(102, 2, 'Mouse'),
(103, 3, 'Keyboard');
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. IN Subquery
8. Is there a customer withis_premium = true?
9. Nested Loop Join (NLJ)
10. No Matching Rows
11. JOIN
12. Join Cardinality Calculation
13. Are there many matchingrows?
14. Nested Loop Join (NLJ)
15. Hash Join
16. EXISTS Subquery
17. Does a customer withis_premium = true exist?
18. Index Seek or NLJ
19. No Matching Rows

