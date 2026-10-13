---
type: "practice"
difficulty: "Deep"
---

# You added a PERSISTED computed column derived from a function and want to index it. What conditions must the function meet for the column to be indexable?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Scalar vs Table-valued Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/Scalar_vs_Table-valued_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To make a persisted computed column indexable, the underlying function must meet specific criteria. Let's break down these requirements step by step.

Firstly, the function must return a deterministic result based on its inputs. This means that given the same input values, it will always produce the same output. Secondly, the function should not contain any side effects—actions that modify database state or external systems—that could cause inconsistent results across different executions. Lastly, the function shouldn't reference columns from tables involved in the computed column’s definition to avoid circular dependencies.

Now let's delve into each of these conditions in more detail.

**Code:**

```sql
-- Example function that returns a deterministic result and does not have side effects
CREATE FUNCTION dbo.GetAgeFromBirthdate(@birthdate DATE)
RETURNS INT
AS
BEGIN
    RETURN DATEDIFF(YEAR, @birthdate, GETDATE())
END;
```

