---
type: "practice"
difficulty: "Intermediate"
---

# What happens if you use a window aggregate like AVG() OVER () with no PARTITION BY and no ORDER BY?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Ordering Inside and Outside OVER](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Ordering_Inside_and_Outside_OVER.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When using a window function like AVG() over a window without specifying PARTITION BY or ORDER BY, you'll get an error because these clauses are required in SQL to define how the data is divided and ordered within the window. This question primarily tests your understanding of the fundamental principles behind window functions, their syntax, and common pitfalls.

The AVG() function over a window without any partitioning or ordering will fail due to a missing clause that's necessary for defining the scope of the aggregation. Let’s break this down step by step.

