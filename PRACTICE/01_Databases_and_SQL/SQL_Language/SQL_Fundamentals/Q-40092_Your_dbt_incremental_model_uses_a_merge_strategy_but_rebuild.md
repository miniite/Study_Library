---
type: "practice"
difficulty: "Intermediate"
---

# Your dbt incremental model uses a merge strategy but rebuilds full history every run. How would you diagnose why the incremental predicate isn't limiting the source scan and fix the is_incremental block?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Incremental Rollups](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Incremental_Rollups.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To diagnose and fix an issue where your dbt incremental model is not using its is_incremental block effectively, you need to understand how SQL's MERGE statement works in dbt and why it might be rebuilding the full history every run. The key steps are:

Understand the Problem: Confirm that the predicate defined in is_incremental isn't limiting the source scan.
Analyze the Merge Strategy: Ensure your SQL logic correctly identifies what to upsert or delete, and why it's not behaving as expected.
Check for Common Issues: Look out for common mistakes like incorrect timestamp handling, missing unique keys, or misconfigured is_incremental blocks.

Let’s break this down step by step.

