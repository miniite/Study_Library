---
type: "practice"
difficulty: "Deep"
---

# What is an INSTEAD OF trigger and on which object type is it typically used?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Trigger Types](<../../../../TOOLS/Oracle/PL-SQL/Trigger_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

An INSTEAD OF trigger in PL/SQL is a special kind of database trigger that allows you to control what happens when a row-level DML operation (INSERT, UPDATE, DELETE) is issued on a view rather than directly on the base table. This mechanism enables you to write custom logic for these operations and can be particularly useful in scenarios where data needs to be manipulated across multiple tables or complex business rules need enforcement.

#### Triage

The question asks about INSTEAD OF triggers, their purpose, and when they are typically used. The interviewer is probing your understanding of PL/SQL and the use cases where such a trigger would be beneficial. I will assume you have some basic knowledge of PL/SQL and views in relational databases.

#### Solution
1. Understanding the Need for INSTEAD OF Triggers

Traditional triggers are associated with tables, meaning they fire when DML operations (INSERT, UPDATE, DELETE) are performed directly on a table. However, views can be used to present data from one or more base tables in a modified or transformed way. When you try to perform row-level DML on a view, the database engine cannot map these operations directly to the underlying tables. This is where INSTEAD OF triggers come into play.

2. How INSTEAD OF Triggers Work

An INSTEAD OF trigger overrides the default behavior of DML statements issued against a view and allows you to handle these operations using procedural logic defined in PL/SQL. Here’s how it works:

Instead Of: The trigger name starts with "INSTEAD OF" followed by the type(s) of operation (INSERT, UPDATE, DELETE).
Body: Inside the body, you write the custom logic that handles the DML operations.
3. Example Scenario

#### Cons
ider a scenario where you have a view that combines data from multiple tables to provide a unified interface:

SQL
CREATE OR REPLACE VIEW employee_details AS
SELECT e.employee_id, e.first_name, e.last_name, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id;

If you want to enforce certain business rules on this view, such as ensuring that the department name always has a specific format, an INSTEAD OF trigger can be used:

SQL
CREATE OR REPLACE TRIGGER check_department_name_trigger
INSTEAD OF INSERT ON employee_details
FOR EACH ROW
BEGIN
    IF :NEW.department_name NOT LIKE '%TEAM%' THEN
        RAISE_APPLICATION_ERROR(-20999, 'Department name must end with TEAM');
    END IF;
END;
4. Key Points and Common Mistakes:
Why Use INSTEAD OF Triggers? They are useful in scenarios like data warehousing or when you need to enforce complex business rules on views.
Common Mistake: Not considering the FOR EACH ROW clause, which is essential for row-level DML operations.
Best Practices: Always validate the incoming data and handle exceptions gracefully.

#### Closing Summary

INSTEAD OF triggers are a powerful feature in PL/SQL that allow you to control the behavior of DML operations on views. They are particularly useful when dealing with complex views or when you need to enforce business rules across multiple tables through a single view interface.

#### Common Mistakes/Follow-Ups

Question: How do INSTEAD OF triggers differ from regular table-level triggers?

Answer: Regular table-level triggers fire on the base table, while INSTEAD OF triggers override the default behavior and allow you to handle DML operations directly within the trigger body.

Question: Are there any limitations or restrictions when using INSTEAD OF triggers?

Answer: Yes, they are limited to views and do not support row-level security policies. Additionally, if a view has multiple tables, handling complex multi-table logic can be challenging.

**Comparison:**

| Table: Employees | Columns: employee_id, first_name, last_name, department_id |
|------------------|-----------------------------------------------------------|
| Table: Departments | Columns: department_id, department_name                    |

**Code:**

```sql
-- Example of an INSTEAD OF trigger for INSERT on a view
CREATE OR REPLACE TRIGGER check_department_name_trigger
INSTEAD OF INSERT ON employee_details
FOR EACH ROW
BEGIN
    IF :NEW.department_name NOT LIKE '%TEAM%' THEN
        RAISE_APPLICATION_ERROR(-20999, 'Department name must end with TEAM');
    END IF;
END;
```

**Example input / output:**

```text
-- Input to the INSTEAD OF trigger
INSERT INTO employee_details (employee_id, first_name, last_name, department_name)
VALUES (1001, 'John', 'Doe', 'Sales');

-- Expected output (trigger logic will prevent insertion if the name is not valid)
```

**Data:**

```sql
-- Sample view creation
CREATE OR REPLACE VIEW employee_details AS
SELECT e.employee_id, e.first_name, e.last_name, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id;

-- Example of a regular table trigger (for comparison)
CREATE OR REPLACE TRIGGER check_department_name_table_trigger
BEFORE INSERT ON departments
FOR EACH ROW
BEGIN
    IF :NEW.department_name NOT LIKE '%TEAM%' THEN
        RAISE_APPLICATION_ERROR(-20999, 'Department name must end with TEAM');
    END IF;
END;
```

**Flow:**

1. User inserts row into view
2. INSTEAD OF trigger capturesoperation
3. Trigger logic processes data
4. If valid, performs the actualinsert on underlying tables;else, raises error

