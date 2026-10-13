---
type: "practice"
difficulty: "Basic"
---

# What are the three main timing options for a DML trigger and what does each one mean?

**Difficulty:** Basic · **Levels:** Fresher · [↑ DML vs DDL Triggers](<../../../../01_Databases_and_SQL/Database_Objects/Triggers/DML_vs_DDL_Triggers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database development, triggers are special stored procedures that automatically execute in response to specific events on a table or view. The key aspect of this question revolves around understanding the three main timing options available for DML (Data Manipulation Language) triggers and what they signify.

To start, let's break down the three timing options: BEFORE, AFTER, and INSTEAD OF. Each option dictates when the trigger code will be executed relative to the data manipulation operation. This is crucial because it affects how the database processes the transaction and can impact performance and integrity.

**Code:**

```sql
-- Example of a BEFORE trigger
CREATE TRIGGER before_update_trigger
BEFORE UPDATE ON employees
FOR EACH ROW
BEGIN
    -- Trigger code here
END;

-- Example of an AFTER trigger
CREATE TRIGGER after_insert_trigger
AFTER INSERT ON departments
FOR EACH ROW
BEGIN
    -- Trigger code here
END;

-- Example of an INSTEAD OF trigger (rare but useful in certain scenarios)
CREATE TRIGGER instead_of_delete_trigger
INSTEAD OF DELETE ON sales
BEGIN
    -- Trigger code here
END;
```

