---
type: "practice"
difficulty: "Deep"
---

# You have a cursor that must see rows inserted by its own transaction after it opened — which cursor sensitivity setting supports that and what is the cost?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Ref Cursor Consistency and Limits](<../../../../01_Databases_and_SQL/Database_Objects/Cursors/Ref_Cursor_Consistency_and_Limits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to ensure that a cursor sees rows inserted within the same transaction as the cursor itself. The key concept here is "cursor stability" or "transaction isolation level." Specifically, the CURSOR STABILITY option in SQL Server and similar mechanisms in other databases can achieve this requirement. However, it's important to understand that using such settings comes with performance costs.

The question delves into understanding how transaction isolation levels work and their trade-offs, particularly focusing on CURSOR STABILITY. Let’s break this down step by step.

