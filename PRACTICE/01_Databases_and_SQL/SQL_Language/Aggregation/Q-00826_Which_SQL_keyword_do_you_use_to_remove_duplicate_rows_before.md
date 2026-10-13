---
type: "practice"
difficulty: "Basic"
---

# Which SQL keyword do you use to remove duplicate rows before an aggregate counts or sums them, and where does it go in the function call?

**Difficulty:** Basic · **Levels:** Fresher · [↑ GROUP BY and HAVING](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/GROUP_BY_and_HAVING.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with databases, removing duplicates before performing aggregate functions is a common requirement. The SQL keyword used for this purpose is DISTINCT, which ensures that only unique rows are considered in your aggregation. In terms of placement, DISTINCT is typically applied directly within the SELECT clause to filter out duplicate records.

To summarize:

TL;DR: Use DISTINCT before the aggregate function in the SELECT clause.
Placement: The DISTINCT keyword goes immediately after the column or columns from which you want to remove duplicates, right before any aggregate functions like SUM, COUNT, etc.

Let's dive into a detailed explanation and walk through an example:

#### Triaging the Question

The question is essentially asking about how to ensure that rows containing duplicate values are excluded when performing aggregate calculations. This can be important in scenarios where you want accurate results, such as counting unique users or summing distinct sales amounts.

#### Assumptions
We are working with a table that contains potentially duplicated entries.
The goal is to perform an aggregation on the data but exclude any duplicates first.

#### First-Principles Solution

To achieve this, you would use DISTINCT within your SQL query. Here’s how it works:

Placement of DISTINCT: DISTINCT should be placed right after the column or columns from which you want to eliminate duplicates. For instance, if you have a table called Orders and you want to count unique customers who made orders, you would use:
SQL
SELECT COUNT(DISTINCT customer_id) FROM Orders;
Aggregate Function Position: The aggregate function (like COUNT, SUM, etc.) should follow the DISTINCT keyword.

#### Example
Walkthrough

Let's say we have an Orders table with columns: order_id, customer_id, and item_name. If there are multiple orders from the same customer for the same item, our goal is to count each unique combination of customer_id and item_name.

SQL
-- Incorrect query (counts all orders)
SELECT COUNT(*) FROM Orders;

-- Correct query with DISTINCT to ensure uniqueness in the count
SELECT COUNT(DISTINCT CONCAT(customer_id, '-', item_name)) AS UniqueOrdersCount FROM Orders;

In this example:

The first query counts all rows, which may include duplicates.
The second query uses CONCAT and DISTINCT to create a unique identifier for each combination of customer_id and item_name, ensuring that the count is accurate.

#### Closing Summary

To summarize, when you need to remove duplicate values before performing an aggregation in SQL, use the DISTINCT keyword followed by your aggregate function. Place DISTINCT after the column or columns from which you want to eliminate duplicates.

#### Common Mistakes
and Follow-Ups
Mistake 1: Not using DISTINCT at all can lead to incorrect results due to duplicate entries.

#### Follow-Up
What if we have a large table with millions of rows? In such cases, consider indexing the columns used in conjunction with DISTINCT for performance optimization.

**Code:**

```sql
-- Example query without DISTINCT
SELECT COUNT(*) FROM Orders;

-- Corrected query using DISTINCT
SELECT COUNT(DISTINCT customer_id) AS UniqueCustomerCount FROM Orders;
```

