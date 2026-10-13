---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd reconstruct a full agent trace tree from a spans table using a recursive CTE on parent_span_id, and how you cap depth and detect cycles from a buggy planner loop.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Recursive CTEs](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Recursive_CTEs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To reconstruct a full agent trace tree from a spans table, we'll use a recursive Common Table Expression (CTE) to traverse the hierarchy based on parent_span_id. Additionally, we need to implement mechanisms to cap depth and detect cycles, as these are critical for preventing infinite loops or excessive memory usage. Let's break this down step-by-step.

Firstly, it’s important to understand that each span in a trace can have zero or more children spans, identified by the parent_span_id column. The goal is to recursively build the entire tree structure starting from the root spans (those with no parent).

Secondly, we need to ensure that the recursion stops at a certain depth to prevent excessive stack usage and cycles are detected to avoid infinite loops.

