---
type: "practice"
difficulty: "Intermediate"
---

# How do you diagnose a join returning duplicate keys because a supposed unique column isn't actually unique in prod data?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Joins vs Subqueries and Join Types](<../../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Joins_vs_Subqueries_and_Join_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When faced with a situation where a query using a supposed unique key is returning duplicates, it's crucial to diagnose the issue systematically. This problem can arise due to subtle issues like null values or inconsistent data types. Let's walk through the steps to identify and resolve this issue from first principles.

First, we need to understand that the core of the problem lies in ensuring that the column we believe is unique actually contains no duplicates in production data. We'll start by verifying the uniqueness of the column through a straightforward subquery approach.

