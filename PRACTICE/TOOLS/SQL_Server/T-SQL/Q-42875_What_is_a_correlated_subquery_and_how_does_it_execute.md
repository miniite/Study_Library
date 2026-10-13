---
type: "practice"
difficulty: "Intermediate"
---

# What is a correlated subquery and how does it execute?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ CROSS APPLY and OUTER APPLY](<../../../../TOOLS/SQL_Server/T-SQL/CROSS_APPLY_and_OUTER_APPLY.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this explanation, I'll break down what a correlated subquery is in T-SQL and how it executes. We'll start with a brief overview, then dive into the details of its execution and provide an example to illustrate the concept.

TL;DR: A correlated subquery is a type of subquery that references the parent query's table or column(s). It gets executed for each row returned by the outer query, making it useful but potentially resource-intensive.

#### Triaging the Question
The core of this question lies in understanding how T-SQL handles correlated subqueries and their execution. The interviewer may be probing your comprehension of nested queries and their performance implications.

Key Assumptions:

Familiarity with basic SQL concepts.
Understanding of how subqueries work within a query (both correlated and non-correlated).

#### First-Principles Solution
A correlated subquery is essentially a subquery that includes one or more columns from the parent query's result set in its WHERE clause. The subquery executes once for every row returned by the outer query, making it useful for conditions that depend on each row’s values.

For instance, consider the following scenario:

We have two tables: Customers and Orders. We want to find all customers who have placed at least one order with a total value greater than $100.

SQL
SELECT c.CustomerID, c.Name
FROM Customers c
WHERE EXISTS (
    SELECT 1 FROM Orders o
    WHERE o.CustomerID = c.CustomerID AND SUM(o.Total) > 100
);

In this example:

The outer query selects customer details.
The subquery (EXISTS clause) checks if there exists an order with a total value greater than $100 for each customer.

Execution Flow:

For every row in the Customers table, T-SQL evaluates the subquery.
If the subquery returns any rows (indicating that at least one order meets the condition), it includes the customer in the result set.
This process is repeated for each row returned by the outer query.

#### Common Mistakes
and Follow-Ups:

Performance Issues: Correlated subqueries can be very slow because they perform a nested loop join, which involves multiple executions of the subquery. It's crucial to understand when to use JOINs or other optimization techniques.
Incorrect Usage: Using correlated subqueries unnecessarily can lead to complex and hard-to-maintain queries.

#### Closing Summary
Correlated subqueries are powerful but should be used judiciously due to their performance overhead. Understanding how they work helps in writing efficient T-SQL code, especially when dealing with complex conditions that depend on each row of data.

**Comparison:**

| CustomerID | Name |
|------------|------|
| 2          | Bob  |

**Code:**

```sql
SELECT c.CustomerID, c.Name
FROM Customers c
WHERE EXISTS (
    SELECT 1 FROM Orders o
    WHERE o.CustomerID = c.CustomerID AND SUM(o.Total) > 100
);
```

**Example input / output:**

```text
CustomerID | Name
-----------|------
2          | Bob
```

**Data:**

```plaintext
Customers table:
CustomerID | Name
-----------|------
1          | Alice
2          | Bob

Orders table:
OrderID | CustomerID | Total
--------|------------|-------
1       | 1          | 95.00
2       | 1          | 105.00
3       | 2          | 80.00
4       | 2          | 120.00
```

**Flow:**

1. Yes
2. No
3. Customers
4. Is the customer's total ordervalue > $100?
5. Include in result
6. D

