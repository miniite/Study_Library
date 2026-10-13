---
type: "practice"
difficulty: "Intermediate"
---

# What is a trigger in T-SQL and when would you use one?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ T-SQL Basics](<../../../../TOOLS/SQL_Server/T-SQL/T-SQL_Basics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

A trigger in T-SQL is a database object that automatically executes a set of statements or a stored procedure whenever an event occurs, such as an insert, update, or delete operation on a specific table. Triggers are used to enforce business rules and maintain data integrity without having to manually write code every time a change is made.

#### Triaging the Question

The question is asking for an explanation of what a trigger is in T-SQL and when it would be useful. The interviewer might be probing your understanding of database operations, event-driven programming, and how triggers can help maintain data integrity and enforce business rules efficiently.

Detailed Solution:
What is a Trigger?

A trigger in SQL Server (T-SQL) is a special kind of stored procedure that automatically runs when an event occurs on the associated table. Triggers are defined per table or view and can be used to respond to INSERT, UPDATE, DELETE, or SELECT statements.

When to Use a Trigger:

Triggers are particularly useful in scenarios where you need to enforce complex business rules or maintain data integrity without having to manually write code every time a change is made.

Enforcing Business Rules:

#### Example
Ensure that when an employee's job status changes, their related records (e.g., salary adjustments) are updated accordingly.
Maintaining Data Integrity:

#### Example
Prevent deleting a record if it has child records in another table.
Logging Changes:

#### Example
Log all insert and update operations for auditing purposes.
Implementing Complex Business Logic:

#### Example
When an order is placed, automatically deduct the stock level from the inventory.
First Principles Approach:
Define the Event: Determine which operation (INSERT, UPDATE, DELETE) will trigger the execution of the code.
Specify the Scope: Define on which table or view the trigger will operate.
Determine the Action: Write the T-SQL statements that will be executed when the event occurs.

#### Example

#### Cons
ider a scenario where you want to log every update made to the Employee table:

SQL
CREATE TRIGGER trg_Employee_Update ON Employee
AFTER UPDATE AS
BEGIN
    INSERT INTO EmployeeLog (EmployeeID, ColumnName, OldValue, NewValue)
    SELECT
        i.EmployeeID,
        c.column_name,
        deleted.[c.column_name],
        inserted.[c.column_name]
    FROM
        inserted i
        JOIN deleted d ON i.EmployeeID = d.EmployeeID
        CROSS APPLY (
            SELECT [column1] = CASE WHEN i.column1 <> d.column1 THEN i.column1 ELSE NULL END,
                   [column2] = CASE WHEN i.column2 <> d.column2 THEN i.column2 ELSE NULL END,
                   -- Add other columns as needed
            FROM inserted i, deleted d
        ) c;
END;

#### Closing Summary

Triggers are a powerful feature in T-SQL that can help automate complex business rules and maintain data integrity. They allow you to react to specific database events without writing procedural code for each operation.

#### Common Mistakes/Follow-Ups
Ignoring Performance Impact: Triggers can significantly impact performance, especially if they contain complex logic or are fired frequently.
Overusing Triggers: Avoid using triggers for simple tasks that could be handled by simpler constraints or views.
Debugging Challenges: Debugging issues with triggers can be difficult because the execution context is different from a normal stored procedure.

