---
type: "practice"
difficulty: "Deep"
---

# Can aggregate functions be used inside a CASE expression? Give an example.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ NULL vs Zero in Aggregates](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/NULL_vs_Zero_in_Aggregates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In SQL, aggregate functions are typically used to summarize or compute values across a set of rows. A common scenario is using them in SELECT statements to get summary statistics like sums, counts, and averages. However, they can also be utilized within more complex expressions, including the CASE statement. This flexibility allows for conditional logic that incorporates aggregate functions.

#### Triaging the Question

The question asks whether it's possible to use aggregate functions inside a CASE expression. The interviewer is likely testing your understanding of SQL syntax and its capabilities. Key assumptions include:

You are familiar with both CASE expressions and aggregate functions.
You understand how these two constructs can be combined.

#### First-Principles Solution

To answer this question, we need to explore the syntax and behavior of SQL. The basic structure of a CASE expression involves multiple conditions and corresponding values or expressions. Aggregate functions like SUM, COUNT, etc., are used for summarizing data. Combining these two requires careful handling.

#### Example Scenario

Suppose you have a table orders with columns order_id, product_id, and quantity. You want to calculate the total quantity of each product, but only if the total quantity exceeds 100 units.

SQL Example
SQL
SELECT
    product_id,
    CASE
        WHEN SUM(quantity) > 100 THEN 'Yes'
        ELSE 'No'
    END AS is_over_100
FROM
    orders
GROUP BY
    product_id;

In this example:

SUM(quantity) is the aggregate function, which calculates the total quantity for each product_id.
The CASE expression checks if the sum exceeds 100 and returns 'Yes' or 'No' accordingly.
The GROUP BY clause ensures that the aggregation happens per product.

#### Closing Summary
+ Common Mistakes/Follow-Ups

This example demonstrates how aggregate functions can be integrated into conditional logic using a CASE statement. It's important to note that:

Aggregation and Grouping: Ensure that your conditions align with the grouping in the query.

#### Performance Considerations
Combining complex logic like this might impact performance, especially on large datasets.

#### Common Mistakes
Forgetting to include GROUP BY for aggregate functions: This is a common pitfall when mixing CASE and aggregation.
Incorrect condition placement: Placing conditions outside the CASE statement can lead to errors or incorrect results.

If you were asked follow-up questions, they might involve:

Exploring alternatives or optimizations.
Discussing potential performance implications.
Dealing with edge cases where data distribution is uneven.

**Comparison:**

```
tableOrderSummary {
    row 1: "product_id"
    row 2: "is_over_100"
    row 3: "1"
    row 4: "Yes"
    row 5: "2"
    row 6: "No"
}
```

**Code:**

```sql
SELECT
    product_id,
    CASE
        WHEN SUM(quantity) > 100 THEN 'Yes'
        ELSE 'No'
    END AS is_over_100
FROM
    orders
GROUP BY
    product_id;
```

**Example input / output:**

```text
-- Input:
SELECT
    product_id,
    CASE
        WHEN SUM(quantity) > 100 THEN 'Yes'
        ELSE 'No'
    END AS is_over_100
FROM
    orders
GROUP BY
    product_id;

-- Output:
+------------+----------------+
| product_id | is_over_100    |
+------------+----------------+
|          1 | Yes            |
|          2 | No             |
+------------+----------------+
```

**Data:**

```sql
CREATE TABLE orders (
    order_id INT,
    product_id INT,
    quantity INT
);

INSERT INTO orders (order_id, product_id, quantity)
VALUES
(1, 1, 50),
(2, 1, 60),
(3, 2, 75),
(4, 2, 80);
```

**Flow:**

1. SELECT product_id, CASEWHEN SUM(quantity) > 100THEN 'Yes' ELSE 'No' END ASis_over_100
2. GROUP BY product_id?
3. Calculate aggregate and applycase logic

