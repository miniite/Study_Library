---
type: "practice"
difficulty: "Basic"
---

# Can you order or sort results that come from a JOIN? Where does ORDER BY go?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Join Practice Problems](<../../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Join_Practice_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Sure! Let's break down the core of your question and then explore it in detail.

TL;DR:

To order or sort results from a JOIN, you can use an ORDER BY clause at the end of your query, but there are some nuances to consider. Generally, ORDER BY is applied after all joins have been performed. Understanding how joins work and where ORDER BY fits in will help you write more efficient queries.

#### Triaging the Question

The question is asking about ordering results when using a JOIN. Specifically, it wants to know if you can order or sort the output of a JOIN, and if so, where exactly should the ORDER BY clause be placed. This involves understanding how joins combine data from multiple tables and how sorting works within that context.

#### Assumptions
You have two tables, TableA and TableB, which you are joining based on some common column(s).
You want to sort the final result set.
Your database system supports SQL (as most do).
Step-by-Step Solution:
1. Understanding Joins

When using a JOIN, your query combines rows from two or more tables based on a related column between them. For example, you might join TableA and TableB on the CustomerID field:

SQL
SELECT A.*, B.*
FROM TableA AS A
JOIN TableB AS B ON A.CustomerID = B.CustomerID;

This query retrieves all columns from both tables for each matching row.

2. Applying ORDER BY After JOIN

Once the join has been performed, the combined result set can be sorted using ORDER BY. For instance:

SQL
SELECT A.*, B.*
FROM TableA AS A
JOIN TableB AS B ON A.CustomerID = B.CustomerID
ORDER BY A.CustomerID;

Here, the ORDER BY clause is placed after the join and ensures that the result set is sorted by CustomerID.

3. Alternative Placement of ORDER BY

If you place the ORDER BY before the JOIN, it will only affect one of the tables involved in the join:

SQL
SELECT A.*, B.*
FROM (SELECT * FROM TableA WHERE CustomerID = 1) AS A
JOIN TableB ON A.CustomerID = B.CustomerID
ORDER BY A.CustomerID;

In this case, ORDER BY is applied only to the result of the subquery on TableA.

4. Using Common Joins (INNER, LEFT, RIGHT)

The placement of ORDER BY remains consistent regardless of the type of join:

SQL
-- INNER JOIN
SELECT A.*, B.*
FROM TableA AS A
INNER JOIN TableB AS B ON A.CustomerID = B.CustomerID
ORDER BY A.CustomerID;

-- LEFT JOIN
SELECT A.*, B.*
FROM TableA AS A
LEFT JOIN TableB AS B ON A.CustomerID = B.CustomerID
ORDER BY A.CustomerID;

#### Closing Summary

To summarize, the ORDER BY clause should be placed after all joins in your SQL query. It ensures that the combined result set is sorted as intended. Placing it earlier can have unintended effects or even be ignored by the database depending on its implementation.

#### Common Mistakes
and Follow-Ups
Mistake: Placing ORDER BY before a join, leading to incorrect sorting.

#### Follow-Up
Explain how different RDBMS handle ORDER BY differently (e.g., MySQL might optimize out early ORDER BY clauses).

#### Example
Provide an example where ORDER BY is applied in a subquery versus directly after the join.

**Code:**

```sql
-- Example of ordering after join
SELECT A.*, B.*
FROM TableA AS A
JOIN TableB AS B ON A.CustomerID = B.CustomerID
ORDER BY A.CustomerID;
```

