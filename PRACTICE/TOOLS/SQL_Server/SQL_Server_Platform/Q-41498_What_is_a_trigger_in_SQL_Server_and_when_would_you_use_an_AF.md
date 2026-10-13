---
type: "practice"
difficulty: "Intermediate"
---

# What is a trigger in SQL Server and when would you use an AFTER trigger?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ MERGE and Upserts](<../../../../TOOLS/SQL_Server/SQL_Server_Platform/MERGE_and_Upserts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Triggers in SQL Server are special types of stored procedures that automatically execute in response to certain events, such as INSERT, UPDATE, or DELETE operations on a table. They allow developers to enforce business rules and maintain data integrity without writing complex application logic.

The key idea here is that triggers provide a way to intercept database actions and modify their behavior before or after the action occurs. Let's break down when you would use an AFTER trigger:

#### Triaging the Question
Question: What is an AFTER trigger in SQL Server, and when would you use it?

#### Assumptions
We assume familiarity with basic SQL concepts like INSERT, UPDATE, and DELETE operations.
What’s Being Probed: The interviewer might be looking for your understanding of transactional integrity and the ability to enforce rules post-operation.
Detailed Solution

An AFTER trigger is executed after a specified data modification event (INSERT, UPDATE, or DELETE) has completed. This means that any operations within the AFTER trigger will only execute if the original operation succeeds. Here’s how you can reason through when to use an AFTER trigger:

Post-Operation Data Integrity Checks:

Imagine a scenario where you need to ensure that after an INSERT into a table, a corresponding entry is made in another related table. You could write an AFTER INSERT trigger on the first table to handle this.

Auditing and Logging:

After every update or delete operation, it might be necessary to log changes for auditing purposes. An AFTER trigger can help capture these details.

Business Rules Enforcement:

Post-operation business rules are another use case. For example, after an INSERT, you may want to ensure that the data meets specific conditions (e.g., checking if a price change has been made on weekends).

Transaction Management:

AFTER triggers can be used for transactional post-processing tasks. If an operation fails and rolls back, the AFTER trigger won't execute.

#### Example Scenario

Let’s consider a simple example where we have two tables: Employees and EmployeeHistory. We want to log every change in employee information into the EmployeeHistory table after any update or delete operation on the Employees table.

SQL
-- Create the EmployeeHistory table
CREATE TABLE EmployeeHistory (
    Id INT PRIMARY KEY IDENTITY,
    OldEmployeeName NVARCHAR(255),
    OldDepartment NVARCHAR(100),
    ChangedOn DATETIME DEFAULT GETDATE(),
    Action NVARCHAR(6) CHECK (Action IN ('INSERT', 'UPDATE', 'DELETE'))
);

-- Create an AFTER trigger
CREATE TRIGGER trg_AfterUpdateOrDeleteEmployee
ON Employees
AFTER UPDATE, DELETE
AS
BEGIN
    -- Log the old values for deleted or updated records
    IF EXISTS (SELECT 1 FROM DELETED)
        INSERT INTO EmployeeHistory (OldEmployeeName, OldDepartment)
        SELECT Name, Department FROM DELETED;

    -- For updates, insert both the old and new values
    IF EXISTS (SELECT 1 FROM INSERTED)
        INSERT INTO EmployeeHistory (OldEmployeeName, OldDepartment)
        SELECT i.Name, d.Department
        FROM INSERTED i JOIN DELETED d ON i.Id = d.Id;
END;

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, AFTER triggers are useful for post-operation data integrity checks and logging. However, they should be used judiciously as excessive use can slow down database performance.

#### Common Mistakes
to Avoid:

Overusing AFTER triggers can lead to performance issues due to the overhead of additional operations.
Not considering the transactional context (i.e., ensuring that you only log changes when transactions are committed).

If asked a follow-up, discuss potential trade-offs between using triggers and alternative solutions like application-level validation or constraint-based approaches.

**Comparison:**

```
ID	OldEmployeeName	OldDepartment	ChangedOn	Action
1	John Doe	HR	[Current Date]	UPDATE
ID	OldEmployeeName	OldDepartment	ChangedOn	Action
2	John Doe	IT	[Current Date]	UPDATE
```

**Code:**

```sql
CREATE TABLE EmployeeHistory (
    Id INT PRIMARY KEY IDENTITY,
    OldEmployeeName NVARCHAR(255),
    OldDepartment NVARCHAR(100),
    ChangedOn DATETIME DEFAULT GETDATE(),
    Action NVARCHAR(6) CHECK (Action IN ('INSERT', 'UPDATE', 'DELETE'))
);

CREATE TRIGGER trg_AfterUpdateOrDeleteEmployee
ON Employees
AFTER UPDATE, DELETE
AS
BEGIN
    IF EXISTS (SELECT 1 FROM DELETED)
        INSERT INTO EmployeeHistory (OldEmployeeName, OldDepartment)
        SELECT Name, Department FROM DELETED;

    IF EXISTS (SELECT 1 FROM INSERTED)
        INSERT INTO EmployeeHistory (OldEmployeeName, OldDepartment)
        SELECT i.Name, d.Department
        FROM INSERTED i JOIN DELETED d ON i.Id = d.Id;
END;
```

**Example input / output:**

```text
-- Update an employee's department
UPDATE Employees SET Department = 'IT' WHERE Id = 1;

-- Check EmployeeHistory table
SELECT * FROM EmployeeHistory;
```

**Data:**

```sql
CREATE TABLE Employees (
    Id INT PRIMARY KEY,
    Name NVARCHAR(255),
    Department NVARCHAR(100)
);

-- Sample data
INSERT INTO Employees (Id, Name, Department) VALUES (1, 'John Doe', 'HR');
```

**Flow:**

1. Yes
2. Start Transaction
3. Insert or Update Operation onEmployees Table
4. Check if Trigger Exists
5. Execute AFTER Trigger
6. Log Changes inEmployeeHistory Table
7. End Transaction

