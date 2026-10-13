---
type: "practice"
difficulty: "Basic"
---

# How do you handle a case where you need data from two tables but only some rows match — which join do you pick?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Joins vs Subqueries and Join Types](<../../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Joins_vs_Subqueries_and_Join_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we are dealing with a situation where we want to retrieve data from two tables, but not every row in both tables has a matching entry. This requires understanding how different types of joins work and choosing the appropriate one based on our requirements.

The key idea here is that each join type (INNER JOIN, LEFT JOIN, RIGHT JOIN, FULL OUTER JOIN) handles unmatched rows differently:

INNER JOIN: Only returns rows where there is a match in both tables.
LEFT JOIN (or LEFT OUTER JOIN): Returns all rows from the left table and matching rows from the right table. If no match is found, NULL values are returned for columns of the right table.
RIGHT JOIN (or RIGHT OUTER JOIN): Similar to LEFT JOIN but returns all rows from the right table with matching rows from the left table. If there's no match in the left table, NULLs will be filled in.
FULL OUTER JOIN: Returns every row that exists in either of the tables and fills in NULL for columns where there is no match.

To determine which join to use, we need to consider the requirements:

Do you only want rows with matches? Use INNER JOIN.
Do you want all rows from one table and matching rows from another? Use LEFT JOIN or RIGHT JOIN depending on which table's full row set is desired.
Do you want both tables to have their full rows, even if there are no matches in the other? Use FULL OUTER JOIN.

Let's break down an example:

Suppose we have two tables: employees and departments. We need data from both but some employees don't belong to any department yet.

