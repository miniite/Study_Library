---
type: "practice"
difficulty: "Basic"
---

# When you write SELECT department, COUNT(*) but forget the GROUP BY, what error or behaviour do you expect in a strict SQL mode?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ COUNT Semantics](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/COUNT_Semantics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When writing SELECT department, COUNT(*) without including a GROUP BY clause, you can anticipate an error or unexpected behavior depending on how your database system enforces SQL standards. In a strict SQL mode, this query will likely result in an error due to the violation of the SQL standard.

In essence, the absence of a GROUP BY clause means that COUNT(*) is intended to operate over all rows without any grouping. However, the SQL standard requires that non-aggregated columns (like department here) must be included either as part of an aggregate function or within a GROUP BY clause.

Let's break this down step by step:

Understanding the Error: The database expects explicit instructions on how to group rows for aggregation purposes. Without a GROUP BY, it doesn't know whether you intended to count all rows together, across departments, or in some other way.
Expected Behavior: In most SQL implementations with strict mode enabled (e.g., PostgreSQL), this query will raise an error message such as "Column 'department' is invalid for aggregation" or a similar error indicating that the GROUP BY clause is missing.

Here's how you would typically resolve this issue:

SQL
SELECT department, COUNT(*)
FROM employees
GROUP BY department;

This corrected version groups rows by each department, and then counts the number of records in each group. If strict mode isn't enabled or if there are no specific rules against such queries (which is not standard), some databases might still execute it but with a warning.

