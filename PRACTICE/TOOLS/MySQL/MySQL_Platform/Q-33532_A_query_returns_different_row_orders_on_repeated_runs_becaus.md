---
type: "practice"
difficulty: "Intermediate"
---

# A query returns different row orders on repeated runs because there's no ORDER BY on a GROUP BY — how do you explain and guarantee deterministic ordering?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Ordering and Limits](<../../../../TOOLS/MySQL/MySQL_Platform/Ordering_and_Limits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this issue, we need to understand why MySQL behaves differently when running the same query multiple times without an ORDER BY clause in conjunction with a GROUP BY statement. The core of the problem lies in how MySQL processes and returns results from such queries. Let's break it down step by step.

First, let’s state what the question is really asking: We need to explain why the row order changes when we run a query without an explicit ORDER BY clause after performing a GROUP BY. Additionally, we will explore how to guarantee deterministic ordering in such cases.

#### Triage

The key assumptions here are:

The query involves a GROUP BY clause.
There is no ORDER BY clause specified.
The results are returned in an order that can differ between runs.

To ensure consistent results, we need to understand how MySQL handles the lack of an ORDER BY clause and then implement a mechanism to enforce deterministic ordering.

#### First Principles Solution

MySQL does not guarantee any specific row order when a query returns multiple rows without an explicit ORDER BY clause. This is because the order can vary based on several factors, such as internal optimizations, execution plans, or even changes in system state. To achieve consistent results, we need to explicitly specify the desired ordering using the ORDER BY clause.

Step-by-Step Solution
Identify Columns for Ordering: Determine which columns are relevant and should dictate the order of rows.
Add ORDER BY Clause: Include an ORDER BY clause in your query to ensure deterministic results.
Choose Sorting Order: Decide whether ascending (ASC) or descending (DESC) sorting is required based on business logic.

Here’s how you can modify a query:

SQL
SELECT column1, SUM(column2) AS total
FROM my_table
GROUP BY column1
ORDER BY total DESC;

This ensures that the results are consistently ordered by total in descending order.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, to guarantee deterministic ordering when using GROUP BY without an explicit ORDER BY, always include the ORDER BY clause with the appropriate columns and sorting direction. Failure to do so can lead to inconsistent results due to MySQL’s internal processing mechanisms.

Some common mistakes and follow-ups might include:

Ignoring ORDER BY: Not realizing that it is necessary for consistent results.
Incorrect ORDER BY Clause: Using irrelevant or incorrectly ordered columns, which could affect performance or result interpretation.

#### Performance Considerations
Ensuring the ORDER BY clause does not significantly impact query performance, especially on large datasets.

If asked to provide an example, you might consider a schema and sample data:

SQL
CREATE TABLE sales (
    id INT PRIMARY KEY,
    product_id INT,
    quantity INT
);

INSERT INTO sales (id, product_id, quantity) VALUES
(1, 101, 5),
(2, 102, 3),
(3, 101, 7),
(4, 102, 6);

With this schema and data:

SQL
SELECT product_id, SUM(quantity) AS total_quantity
FROM sales
GROUP BY product_id
ORDER BY total_quantity DESC;

This query will consistently return the products ordered by their total quantity in descending order.

**Code:**

```sql
CREATE TABLE sales (
    id INT PRIMARY KEY,
    product_id INT,
    quantity INT
);

INSERT INTO sales (id, product_id, quantity) VALUES
(1, 101, 5),
(2, 102, 3),
(3, 101, 7),
(4, 102, 6);
```

**Example input / output:**

```text
SELECT product_id, SUM(quantity) AS total_quantity
FROM sales
GROUP BY product_id
ORDER BY total_quantity DESC;

-- Example Output:
-- +------------+----------------+
-- | product_id | total_quantity |
-- +------------+----------------+
-- |        101 |             12 |
-- |        102 |              9 |
-- +------------+----------------+
```

