---
type: "practice"
difficulty: "Basic"
---

# On which DML operations can you define a trigger in SQL?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ DML vs DDL Triggers](<../../../../01_Databases_and_SQL/Database_Objects/Triggers/DML_vs_DDL_Triggers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Triggers are database objects that automatically execute when a specific event, such as an insert, update, or delete operation, occurs on a particular table. In SQL, we can define triggers to enforce constraints, maintain data integrity, and perform other actions at the time of a DML (Data Manipulation Language) operation.

To fully answer this question, let's first clarify what DML operations are and then discuss which ones allow for trigger definition.

**Code:**

```sql
-- Example DML operations
INSERT INTO table_name VALUES (...);
UPDATE table_name SET column1 = value1 WHERE condition;
DELETE FROM table_name WHERE condition;
```

