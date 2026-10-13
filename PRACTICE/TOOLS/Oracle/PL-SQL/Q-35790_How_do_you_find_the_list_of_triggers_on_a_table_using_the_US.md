---
type: "practice"
difficulty: "Intermediate"
---

# How do you find the list of triggers on a table using the USER_TRIGGERS data dictionary view?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Trigger Types](<../../../../TOOLS/Oracle/PL-SQL/Trigger_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To find the list of triggers on a specific table using the USER_TRIGGERS data dictionary view in PL/SQL, you need to query this view and filter based on the table name. This is a foundational task that helps database developers understand how tables interact with other objects like triggers.

The question is probing your familiarity with PL/SQL and Oracle's metadata views. Let’s break it down step by step:

Understand the USER_TRIGGERS View: The USER_TRIGGERS view contains information about all triggers owned by the current user.
Filtering Triggers for a Specific Table: You need to filter the results based on the table name you are interested in.

Now, let's proceed with the first-principles solution:

SQL
SELECT TRIGGER_NAME, TRIGGER_SCHEMA, TABLE_NAME
FROM USER_TRIGGERS
WHERE TABLE_NAME = 'YOUR_TABLE_NAME';

Replace 'YOUR_TABLE_NAME' with the actual name of your table. This query will return a list of triggers associated with that specific table.

**Code:**

```sql
SELECT TRIGGER_NAME, TRIGGER_SCHEMA, TABLE_NAME
FROM USER_TRIGGERS
WHERE TABLE_NAME = 'EMPLOYEES';
```

