---
type: "practice"
difficulty: "Intermediate"
---

# How would you find employees who share the same salary using GROUP BY and HAVING?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Basic Query Practice](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Basic_Query_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To find employees who share the same salary in PostgreSQL, we need to use GROUP BY and HAVING. The goal is to group all rows by their salary and then filter out those groups that have more than one employee with the same salary.

First, let's break down what each part of the query does:

GROUP BY: This clause aggregates the records into groups based on a specified column (in this case, salary).
HAVING: This is used to filter these groups. Unlike WHERE, which filters rows before grouping, HAVING filters after grouping.

Now, let's construct our query step by step.

