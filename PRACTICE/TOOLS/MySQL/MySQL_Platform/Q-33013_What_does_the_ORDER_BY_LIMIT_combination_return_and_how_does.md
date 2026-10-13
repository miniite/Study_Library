---
type: "practice"
difficulty: "Basic"
---

# What does the ORDER BY ... LIMIT combination return and how does adding DESC change it?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Ordering and Limits](<../../../../TOOLS/MySQL/MySQL_Platform/Ordering_and_Limits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In MySQL, the ORDER BY clause is used to sort rows based on one or more columns, while LIMIT restricts the number of rows returned. When you combine these two clauses, you get a result set that is both sorted and truncated to a specified number of rows. Adding DESC (for descending) after a column in the ORDER BY clause reverses the sorting order.

To better understand this concept, let's break down the query step by step from first principles.

