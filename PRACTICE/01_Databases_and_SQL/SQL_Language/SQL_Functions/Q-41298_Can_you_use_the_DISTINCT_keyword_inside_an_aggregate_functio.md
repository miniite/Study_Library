---
type: "practice"
difficulty: "Intermediate"
---

# Can you use the DISTINCT keyword inside an aggregate function such as COUNT(DISTINCT column)? Explain.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ NULL vs Zero in Aggregates](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/NULL_vs_Zero_in_Aggregates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: In SQL, using DISTINCT with an aggregate function like COUNT() is valid and can be used to count unique values in a specific column. However, it's important to understand how this operation works internally and the potential performance implications.

The question asks whether we can use the DISTINCT keyword inside an aggregate function such as COUNT(DISTINCT column). Let's break down the concept step by step.

#### Triaging the Question

Core of the Question: The core of the question is to understand if and how DISTINCT can be used with aggregate functions like COUNT(), and what the implications are in terms of performance and functionality.

What's Being Probed:

Knowledge of SQL syntax and semantics.
Understanding of how data aggregation works internally.
Awareness of potential performance issues.

#### Assumptions
We assume a standard relational database system that supports SQL standards (e.g., MySQL, PostgreSQL).
The column in question contains duplicate values.
Step-by-Step Solution

Understanding COUNT(DISTINCT column):

This syntax is valid and is used to count the number of unique values in the specified column.
Internally, SQL first applies the DISTINCT operation on the column, then counts the resulting distinct set.

#### Example
Query:

SQL
SELECT COUNT(DISTINCT customer_id) AS unique_customers FROM orders;

This query will count the number of unique customer_ids in the orders table.

#### Performance Considerations

Using DISTINCT can have performance implications, especially on large tables.
SQL engines may need to perform a sort operation or use additional resources to compute the distinct values before counting.

Alternative Approach:

If you only need to count unique values and not all columns, consider using a subquery:
SQL
SELECT COUNT(*) FROM (SELECT DISTINCT customer_id FROM orders) AS unique_customers;
This approach can be more efficient in some cases as it avoids the full table scan.

#### Closing Summary

In summary, COUNT(DISTINCT column) is valid SQL syntax and can be used to count unique values. However, understanding its performance implications and choosing appropriate queries based on your specific needs can help optimize your database operations.

#### Common Mistakes/Follow-Ups
Incorrect Usage: Mixing DISTINCT with other aggregate functions in a single query without proper grouping can lead to errors.
Performance Issues: Not considering the potential overhead of distinct calculations, especially on large datasets.
Alternative Optimizations: Failing to explore alternative methods like subqueries for performance improvement.

**Code:**

```sql
-- Example Query
SELECT COUNT(DISTINCT customer_id) AS unique_customers FROM orders;
```

