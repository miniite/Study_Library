---
type: "practice"
difficulty: "Deep"
---

# Show me how you'd find the latest row per group and compare the window-function approach against a correlated subquery on cost.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Rewriting Subqueries as Joins](<../../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Rewriting_Subqueries_as_Joins.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To find the latest row per group, we can use both a window function (like ROW_NUMBER()) and a correlated subquery approach. The window function is generally more efficient and easier to understand, whereas the correlated subquery might be less performant but still effective. Let's explore each method step by step.

