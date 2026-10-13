---
type: "practice"
difficulty: "Basic"
---

# What is the GROUP BY clause and how does it work with aggregate functions in Oracle?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Oracle Aggregate Functions](<../../../../TOOLS/Oracle/Oracle_Platform/Oracle_Aggregate_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The GROUP BY clause in Oracle is a fundamental tool used to aggregate data based on specific columns, making it easier to analyze large datasets by summarizing them into more manageable groups. Essentially, when you use GROUP BY, you're instructing the database to group rows that have the same values in specified columns and then perform aggregate functions (like SUM, AVG, COUNT, etc.) on these grouped data.

#### Triage
What is Being Probed

The question is asking for a basic understanding of how the GROUP BY clause works with aggregate functions. The interviewer wants to ensure you know its purpose, syntax, and some common use cases. Assumptions include that you are familiar with SQL basics and can explain concepts clearly.

#### First-Principles Solution
Purpose: GROUP BY is used to divide the result-set of a query into groups based on one or more columns.
Syntax: The basic syntax looks like this:
SQL
SELECT column, aggregate_function(column)
FROM table_name
GROUP BY column;
Working Mechanism:
You specify one or more columns to group by in the GROUP BY clause.
For each unique combination of values in these columns, Oracle performs the specified aggregate function(s).

#### Example
Suppose you have a table sales with columns product_id, quantity_sold, and price. To find the total sales for each product:
SQL
SELECT product_id, SUM(quantity_sold * price) AS total_sales
FROM sales
GROUP BY product_id;
Edge Cases: Ensure that all non-aggregated columns used in the SELECT clause are included in the GROUP BY clause.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, GROUP BY is a powerful feature for summarizing data in Oracle SQL by grouping rows based on specific criteria and applying aggregate functions. A common mistake is forgetting to include all non-aggregated columns in the SELECT clause within the GROUP BY. Follow-up questions might explore more complex use cases involving multiple tables or nested queries.

**Code:**

```sql
-- Example Query
SELECT product_id, SUM(quantity_sold * price) AS total_sales
FROM sales
GROUP BY product_id;
```

