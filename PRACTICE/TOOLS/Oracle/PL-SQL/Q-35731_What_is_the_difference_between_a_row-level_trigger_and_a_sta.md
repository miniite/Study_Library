---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between a row-level trigger and a statement-level trigger?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Trigger Types](<../../../../TOOLS/Oracle/PL-SQL/Trigger_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the difference between row-level triggers and statement-level triggers in PL/SQL, we need to first grasp their core concepts and then delve into how they are triggered and used.

Row-level triggers fire whenever a single row is inserted, updated, or deleted from a table. They provide fine-grained control over database operations at the individual record level, making them suitable for complex business logic that depends on specific rows. Statement-level triggers, on the other hand, are executed only once per statement (INSERT, UPDATE, DELETE), making them ideal for simpler tasks like logging or auditing entire actions.

To illustrate these differences, let's consider a scenario where we need to track changes in employee data and maintain a history of modifications.

**Code:**

```

-- Example table CREATE TABLE Employees ( id INT PRIMARY KEY, name VARCHAR(100), department VARCHAR(50) );

-- Row-level trigger example CREATE OR REPLACE TRIGGER emp_row_trigger BEFORE INSERT OR UPDATE ON Employees FOR EACH ROW BEGIN -- Business logic here, e.g., updating a version number or logging changes END;

-- Statement-level trigger example CREATE OR REPLACE TRIGGER emp_statement_trigger AFTER INSERT OR DELETE ON Employees DECLARE v_count NUMBER; BEGIN SELECT COUNT(*) INTO v_count FROM deleted; -- For DELETE statement DBMS_OUTPUT.PUT_LINE('Number of rows affected: ' || v_count); END;

CODE
Copy

In the row-level trigger example, the `FOR EACH ROW` clause ensures that the trigger code runs for each row being inserted or updated. In contrast, the statement-level trigger is activated only once after a batch operation (INSERT, DELETE) and can provide summary information about the entire set of rows affected.

Understanding these differences helps in deciding which type of trigger to use based on the complexity and scope of the task at hand.

```sql
-- Blank for no additional code needed here
```

**Example input / output:**

```

For a better understanding, let's dry-run an example scenario:

Row-level Trigger Example: Suppose we update the name of employee with ID 1 from "John" to "Jonathan".

SQL
Copy
UPDATE Employees SET name = 'Jonathan' WHERE id = 1;

The row-level trigger would execute once for each updated row, potentially performing additional business logic like logging or updating a version number.

Statement-level Trigger Example: Suppose we delete an employee with ID 1.

SQL
Copy
DELETE FROM Employees WHERE id = 1;

The statement-level trigger would run after the deletion and log that one row was affected.
```

**Flow:**

1. YES
2. NO
3. NO
4. INSERT or UPDATE Statement
5. Is FOR EACH ROW?
6. Execute Row-Level Trigger perRow
7. AFTER INSERT/UPDATEStatement
8. AFTER DELETE Statement
9. Is FOR EACH ROW?
10. Execute Statement-LevelTrigger once
11. To summarize, row-level triggers offer granular control by executing for each affected row, making them suitable for detailed business logic. Conversely, statement-level triggers execute after a batch operation and are better suited for broader actions like logging or summarizing the overall impact of a transaction.

