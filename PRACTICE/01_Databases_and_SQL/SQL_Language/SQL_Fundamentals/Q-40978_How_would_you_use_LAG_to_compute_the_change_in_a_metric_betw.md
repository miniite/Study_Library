---
type: "practice"
difficulty: "Intermediate"
---

# How would you use LAG to compute the change in a metric between consecutive readings per host (a simple delta)?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Gaps and Islands](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Gaps_and_Islands.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute the change in a metric between consecutive readings per host using SQL, we can leverage the LAG window function. This approach allows us to access data from the previous row within the same result set, making it ideal for calculating differences over time. Let's break down the steps and solution in detail.

