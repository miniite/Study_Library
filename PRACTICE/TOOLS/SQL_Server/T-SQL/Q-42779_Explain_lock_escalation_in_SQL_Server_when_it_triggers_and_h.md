---
type: "practice"
difficulty: "Deep"
---

# Explain lock escalation in SQL Server, when it triggers, and how you would prevent it from escalating to a full table lock during a bulk update.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ MERGE Pitfalls and Error Handling](<../../../../TOOLS/SQL_Server/T-SQL/MERGE_Pitfalls_and_Error_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Lock escalation is a critical concept in T-SQL that developers need to understand deeply, especially when performing bulk updates or operations on large tables. It occurs when SQL Server determines that it's more efficient to use an exclusive table lock (Schema Stability) instead of multiple row locks, which can significantly impact performance and concurrency. I'll explain the conditions under which lock escalation happens, how it affects your application, and strategies for preventing it from escalating to a full table lock during bulk updates.

**Code:**

```sql
-- Example scenario triggering lock escalation
BEGIN TRANSACTION;

UPDATE Sales.SalesOrderHeader
SET OrderDate = '2023-10-01'
WHERE SalesPersonID = 271;
```

