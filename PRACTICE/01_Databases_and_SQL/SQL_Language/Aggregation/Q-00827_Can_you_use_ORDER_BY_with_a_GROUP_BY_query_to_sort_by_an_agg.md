---
type: "practice"
difficulty: "Intermediate"
---

# Can you use ORDER BY with a GROUP BY query to sort by an aggregated value?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ GROUP BY and HAVING](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/GROUP_BY_and_HAVING.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To answer whether ORDER BY can be used with a GROUP BY query to sort by an aggregated value, we need to understand how these clauses work together and their limitations. In SQL, GROUP BY is used to group rows that have the same values in specified columns into summary rows, while ORDER BY is typically used to order the result set after grouping.

#### Triage

The question is probing your understanding of SQL syntax and its behavior when combining different clauses like GROUP BY and ORDER BY. We need to ensure you know how these work together and any constraints they might have. Let's break it down step by step from first principles.

Step-by-Step Solution

Understanding GROUP BY:

The GROUP BY clause is used to break down a large set of data into smaller, more manageable groups based on the values in one or more columns.
When using GROUP BY, you can apply aggregate functions like SUM, COUNT, AVG, etc., to these groups.

Using Aggregate Functions with ORDER BY:

After grouping the rows and applying an aggregate function, SQL requires a way to order the final result set.
The ORDER BY clause is used after the GROUP BY to sort the aggregated results in ascending or descending order based on one of the columns or expressions.

Combining GROUP BY and ORDER BY:

Yes, you can use ORDER BY with a GROUP BY query to sort by an aggregated value.
To do this, simply include the aggregate function in the SELECT clause along with the column(s) you want to group by, and then specify the ORDER BY clause.

#### Example
Query

Let's consider a simple example. Suppose we have a table called orders with columns customer_id, product_id, and quantity.

SQL
-- Sample query
SELECT customer_id, product_id, SUM(quantity) AS total_quantity
FROM orders
GROUP BY customer_id, product_id
ORDER BY total_quantity DESC;

In this query:

We are grouping the rows by both customer_id and product_id.
The aggregate function SUM(quantity) is used to calculate the total quantity for each group.
Finally, we sort the results based on the aggregated value total_quantity.

#### Common Mistakes
and Follow-Ups
Misunderstanding Grouping: Ensure you understand that grouping by multiple columns means each combination of values will be treated as a separate group.
Ordering Without Aggregation: Remember that ORDER BY can only order based on the values present in the GROUP BY clause or aggregated functions.
Sorting Order: Be aware of the order in which you write the clauses—GROUP BY must come before ORDER BY.

#### Closing Summary

In summary, yes, you can use ORDER BY with a GROUP BY query to sort by an aggregated value. The key is to include the aggregate function in both the SELECT and GROUP BY parts of your SQL statement, followed by an ORDER BY clause that references one or more columns from the GROUP BY clause.

Self-Check

Correctness + completeness confirmed.

**Code:**

```sql
-- Example query
SELECT customer_id, product_id, SUM(quantity) AS total_quantity
FROM orders
GROUP BY customer_id, product_id
ORDER BY total_quantity DESC;
```

