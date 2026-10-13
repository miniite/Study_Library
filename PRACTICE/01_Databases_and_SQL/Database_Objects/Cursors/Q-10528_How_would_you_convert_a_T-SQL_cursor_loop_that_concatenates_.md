---
type: "practice"
difficulty: "Intermediate"
---

# How would you convert a T-SQL cursor loop that concatenates strings into a set-based STRING_AGG?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Cursors vs Set-based SQL](<../../../../01_Databases_and_SQL/Database_Objects/Cursors/Cursors_vs_Set-based_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When converting a T-SQL cursor loop that concatenates strings to use STRING_AGG, we need to understand that STRING_AGG is a set-based approach, which is more efficient and easier to handle compared to cursors. Cursors are used for row-by-row processing, while STRING_AGG processes the entire dataset at once.

The core idea here is to transform the logic from a cursor into a query that leverages STRING_AGG. Let's break down the problem step by step.

