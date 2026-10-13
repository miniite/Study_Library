---
type: "practice"
difficulty: "Deep"
---

# How would you compute both a per-store daily rank and an all-time rank of sales in one pass and prove that the two OVER clauses can share a single sort?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ First Last per Group and Running Totals](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/First_Last_per_Group_and_Running_Totals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute both a per-store daily rank and an all-time rank of sales in one pass, we need to use window functions effectively within SQL. Specifically, we'll leverage the ROW_NUMBER() function with two different OVER clauses—first for ranking by day and store, then for ranking across all time and stores. This approach ensures that the rankings are computed efficiently without needing multiple passes over the data.

