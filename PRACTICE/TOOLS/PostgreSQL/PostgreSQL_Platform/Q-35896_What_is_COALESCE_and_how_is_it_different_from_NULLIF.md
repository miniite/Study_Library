---
type: "practice"
difficulty: "Intermediate"
---

# What is COALESCE and how is it different from NULLIF?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Materialized CTEs](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Materialized_CTEs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

COALESCE and NULLIF are two powerful functions in PostgreSQL that help manage NULL values, but they serve distinct purposes. COALESCE returns the first non-NULL value in a list of arguments, while NULLIF checks if two expressions are equal; if they are, it returns NULL, otherwise, it returns the first argument.

Let's dive into how each function works and their differences from NULL values.

