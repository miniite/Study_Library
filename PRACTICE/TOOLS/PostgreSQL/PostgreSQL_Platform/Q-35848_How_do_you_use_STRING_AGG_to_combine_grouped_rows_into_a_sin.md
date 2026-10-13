---
type: "practice"
difficulty: "Intermediate"
---

# How do you use STRING_AGG to combine grouped rows into a single comma-separated string?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ FIRST_VALUE and LAST_VALUE](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/FIRST_VALUE_and_LAST_VALUE.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To combine grouped rows into a single comma-separated string using STRING_AGG in PostgreSQL, you'll first need to understand how this function works and its usage context. This is particularly useful when you want to aggregate values from multiple rows within the same group. Let's break it down step by step.

#### Triage
Restating the Question

The core of the question is about using STRING_AGG in PostgreSQL. The interviewer might be probing your understanding of SQL aggregation functions, specifically how to handle concatenation across multiple records. We will assume you have a basic understanding of SQL and PostgreSQL, and we’ll focus on practical usage.

#### First-Principles Solution
Understand the Function: STRING_AGG is an aggregate function in PostgreSQL that concatenates strings within a group defined by a GROUP BY clause.
Syntax and Usage:
SQL
SELECT STRING_AGG(column_name, ', ') WITHIN GROUP (ORDER BY column_name) AS aggregated_string
FROM your_table
GROUP BY grouping_column;

#### Example Scenario
Suppose you have an orders table with columns order_id, customer_id, and product. You want to list the products purchased by each customer in a single comma-separated string.
Step-by-Step Solution

Identify the Table and Columns:

SQL
-- Example schema for orders table
CREATE TABLE orders (
    order_id INT,
    customer_id INT,
    product VARCHAR(255)
);

Write the Query: To get a list of products purchased by each customer, you can use STRING_AGG as follows:

SQL
SELECT
    customer_id,
    STRING_AGG(product, ', ') WITHIN GROUP (ORDER BY product) AS products_purchased
FROM orders
GROUP BY customer_id;

Explanation:

The GROUP BY customer_id clause groups the records by each customer.
STRING_AGG(product, ', ') WITHIN GROUP (ORDER BY product) concatenates all product names for a given customer, with commas and spaces separating them.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Closing: The STRING_AGG function is powerful for combining multiple rows into a single string. Always ensure you use the correct syntax and pay attention to the ordering within the aggregation.

#### Common Mistakes
Forgetting to include an ORDER BY clause, which can lead to non-deterministic results.
Not using WITHIN GROUP, which is required when using STRING_AGG.

#### Example
Data
SQL
-- Sample data for orders table
INSERT INTO orders (order_id, customer_id, product)
VALUES
    (1, 101, 'Laptop'),
    (2, 101, 'Mouse'),
    (3, 102, 'Keyboard'),
    (4, 102, 'Monitor');
Dry-Run
SQL
-- Running the query with sample data
SELECT
    customer_id,
    STRING_AGG(product, ', ') WITHIN GROUP (ORDER BY product) AS products_purchased
FROM orders
GROUP BY customer_id;

This will output:

customer_id	products_purchased
101	Laptop, Mouse
102	Keyboard, Monitor
Self-Check

The solution is correct and complete.

**Comparison:**

|customer_id |products_purchased         |
|------------|--------------------------|
|101         |Laptop, Mouse              |
|102         |Keyboard, Monitor          |

**Code:**

```sql
-- Example schema for orders table
CREATE TABLE orders (
    order_id INT,
    customer_id INT,
    product VARCHAR(255)
);

-- Sample data for orders table
INSERT INTO orders (order_id, customer_id, product)
VALUES
    (1, 101, 'Laptop'),
    (2, 101, 'Mouse'),
    (3, 102, 'Keyboard'),
    (4, 102, 'Monitor');

-- Query to combine products purchased by each customer
SELECT
    customer_id,
    STRING_AGG(product, ', ') WITHIN GROUP (ORDER BY product) AS products_purchased
FROM orders
GROUP BY customer_id;
```

**Example input / output:**

```text
-- Running the query with sample data
SELECT
    customer_id,
    STRING_AGG(product, ', ') WITHIN GROUP (ORDER BY product) AS products_purchased
FROM orders
GROUP BY customer_id;

-- Output:
customer_id |products_purchased         |
------------|--------------------------|
101         |Laptop, Mouse              |
102         |Keyboard, Monitor          |
```

**Data:**

```sql
-- Sample data for orders table
INSERT INTO orders (order_id, customer_id, product)
VALUES
    (1, 101, 'Laptop'),
    (2, 101, 'Mouse'),
    (3, 102, 'Keyboard'),
    (4, 102, 'Monitor');
```

**Flow:**

1. Yes
2. No
3. Start
4. Is there a GROUP BY clause?
5. Use STRING_AGG with ORDERBY
6. String concatenation for eachgroup
7. Invalid query, fix GROUP BYclause
8. End

