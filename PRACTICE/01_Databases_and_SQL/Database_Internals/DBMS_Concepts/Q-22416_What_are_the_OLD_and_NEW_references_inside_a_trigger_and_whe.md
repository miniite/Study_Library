---
type: "practice"
difficulty: "Intermediate"
---

# What are the OLD and NEW references inside a trigger and when is each available?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Lock Granularity](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Lock_Granularity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database systems, triggers play a crucial role in maintaining data integrity and consistency by automatically executing predefined actions upon certain events. The key distinction between OLD and NEW references within a trigger lies in their availability during different phases of the transaction. This article will explain when each reference is available, using clear examples to illustrate practical scenarios.

TL;DR
OLD Reference: Available only for DELETE and UPDATE operations.
NEW Reference: Available exclusively for INSERT and UPDATE operations.
The OLD and NEW references are used within the trigger body to access the row(s) being affected by the triggering event, allowing for complex data manipulation logic.

#### Triaging the Question

The question is essentially asking about the availability of two special reference variables (OLD and NEW) in database triggers. These variables hold information specific to rows that are undergoing changes due to INSERT, UPDATE, or DELETE operations. The interviewer might be probing your understanding of how these references work within different transaction phases and their practical implications.

#### First Principles Solution

Understanding the Context:

Triggers are typically defined on tables.
They can be activated by specific events such as INSERT, UPDATE, or DELETE.

OLD Reference:

The OLD reference is only available for operations that modify existing rows, i.e., UPDATE and DELETE.
In a DELETE trigger, OLD holds the data of the row being deleted.
In an UPDATE trigger, OLD contains the original values before any changes are made.

NEW Reference:

The NEW reference is only available for operations that insert new rows or update existing ones, i.e., INSERT and UPDATE.
In a DELETE trigger, there is no NEW reference because no row is being inserted.
In an UPDATE trigger, NEW contains the updated values of the row after any changes are made.

#### Example Scenario
s

Let's consider a simple table employees and triggers for different operations:

SQL
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    name VARCHAR(50),
    salary DECIMAL(10, 2)
);
DELETE Trigger Example:

A trigger that logs the deletion of an employee:

SQL
DELIMITER $$

CREATE TRIGGER before_employee_delete BEFORE DELETE ON employees
FOR EACH ROW
BEGIN
    INSERT INTO deleted_employees (emp_id, name, salary) VALUES (OLD.emp_id, OLD.name, OLD.salary);
END$$

DELIMITER ;

Here, the OLD reference is used to store the data of the row being deleted.

UPDATE Trigger Example:

A trigger that logs the update of an employee’s salary:

SQL
DELIMITER $$

CREATE TRIGGER before_employee_update BEFORE UPDATE ON employees
FOR EACH ROW
BEGIN
    IF NEW.salary <> OLD.salary THEN
        INSERT INTO updated_employees (emp_id, old_salary, new_salary) VALUES (OLD.emp_id, OLD.salary, NEW.salary);
    END IF;
END$$

DELIMITER ;

In this case, both NEW and OLD references are used to capture the changes before they occur.

#### Closing Summary
+ Common Mistakes
Common Mistake: Assuming that OLD is always available or that NEW can be used in a DELETE trigger.
Key Takeaway: Understanding the context of each operation (INSERT, UPDATE, DELETE) and knowing which references are relevant for specific operations ensures correct usage.
Practical Considerations
Transaction Management: Ensure that your triggers handle concurrency issues appropriately to avoid race conditions or deadlocks.
Data Integrity: Proper use of OLD and NEW helps maintain consistency by allowing you to track changes and enforce business rules effectively.

**Comparison:**

| Column     | Type          |
|------------|---------------|
| emp_id     | INT           |
| name       | VARCHAR(50)   |
| salary     | DECIMAL(10, 2) |
| Column     | Type         |
|------------|--------------|
| emp_id     | INT          |
| old_salary | DECIMAL(10, 2) |
| new_salary | DECIMAL(10, 2) |
| Column     | Type          |
|------------|---------------|
| emp_id     | INT           |
| name       | VARCHAR(50)   |
| salary     | DECIMAL(10, 2) |

**Code:**

```sql
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    name VARCHAR(50),
    salary DECIMAL(10, 2)
);

DELIMITER $$

CREATE TRIGGER before_employee_delete BEFORE DELETE ON employees
FOR EACH ROW
BEGIN
    INSERT INTO deleted_employees (emp_id, name, salary) VALUES (OLD.emp_id, OLD.name, OLD.salary);
END$$

DELIMITER ;

DELIMITER $$

CREATE TRIGGER before_employee_update BEFORE UPDATE ON employees
FOR EACH ROW
BEGIN
    IF NEW.salary <> OLD.salary THEN
        INSERT INTO updated_employees (emp_id, old_salary, new_salary) VALUES (OLD.emp_id, OLD.salary, NEW.salary);
    END IF;
END$$

DELIMITER ;
```

**Data:**

```sql
CREATE TABLE deleted_employees (
    emp_id INT,
    name VARCHAR(50),
    salary DECIMAL(10, 2)
);

CREATE TABLE updated_employees (
    emp_id INT,
    old_salary DECIMAL(10, 2),
    new_salary DECIMAL(10, 2)
);
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. CREATE TRIGGER
8. Triggers Activated
9. DELETE Event Occurs
10. OLD Reference Available?
11. INSERT LOG OF DELETED ROW
12. No Action
13. UPDATE Event Occurs
14. NEW and OLD ReferencesAvailable?
15. LOG UPDATED ROW CHANGES
16. No Action
17. INSERT Event Occurs
18. NEW Reference Available?
19. NO NEW REFERENCELOGGING NEEDED
20. No Action

