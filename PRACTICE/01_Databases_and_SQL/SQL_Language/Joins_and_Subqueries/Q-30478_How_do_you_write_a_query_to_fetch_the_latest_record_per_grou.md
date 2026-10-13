---
type: "practice"
difficulty: "Deep"
---

# How do you write a query to fetch the latest record per group, such as the latest login per user?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Join Practice Problems](<../../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Join_Practice_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To answer this question, we need to understand how to fetch the latest record per group using SQL queries, which is often achieved through a combination of JOIN, SUBQUERY, and window functions or common table expressions (CTEs). Let's break down the problem step by step from first principles.

First, we need to identify what "latest record" means in our context. For this example, let's assume that the latest login per user is determined by the most recent timestamp (login_time). We will explore both a JOIN and SUBQUERY approach to solve this problem.

