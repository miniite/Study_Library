---
type: "practice"
difficulty: "Deep"
---

# How do you find duplicate rows in a table using a self join or subquery?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Rewriting Subqueries as Joins](<../../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Rewriting_Subqueries_as_Joins.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To find duplicate rows in a table, we can use either a self-join or a subquery approach. Both methods leverage the power of SQL to identify and filter out rows that have identical values across different columns. In this explanation, I'll first outline the basic steps for each method before diving into the specific implementation details.

TL;DR: To find duplicates using a self-join, you compare each row with every other row in the table based on certain conditions. With a subquery approach, you use correlated subqueries to check if any other rows match the current row's values.

