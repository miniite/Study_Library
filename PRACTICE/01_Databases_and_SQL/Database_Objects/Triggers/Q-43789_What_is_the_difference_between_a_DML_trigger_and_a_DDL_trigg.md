---
type: "practice"
difficulty: "Basic"
---

# What is the difference between a DML trigger and a DDL trigger?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ DML vs DDL Triggers](<../../../../01_Databases_and_SQL/Database_Objects/Triggers/DML_vs_DDL_Triggers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database systems, triggers are special types of stored procedures that automatically execute in response to certain events on a table or view. Triggers can be further categorized into DML (Data Manipulation Language) triggers and DDL (Data Definition Language) triggers based on the type of event they respond to.

#### Triage

The question asks about the differences between DML and DDL triggers. The interviewer is probing your understanding of the syntax, functionality, and use cases for each type of trigger. We will assume that you are familiar with basic SQL concepts such as tables, views, and SQL statements like INSERT, UPDATE, DELETE.

#### First-Principles Solution

DML Triggers:

Triggered by DML statements: INSERT, UPDATE, or DELETE.
Used for maintaining data integrity, logging changes, auditing operations, or performing complex business logic.

#### Example
A trigger that logs every time a row is updated in the employees table.

DDL Triggers:

Triggered by DDL statements: CREATE, ALTER, DROP.
Used for maintaining schema integrity, ensuring that certain operations are performed before or after schema changes.

#### Example
A trigger that automatically creates a backup database when the employees table is altered.

#### Closing Summary
+ Common Mistakes/Follow-Ups

DML triggers respond to data manipulation activities (like insertions and updates), while DDL triggers handle schema changes. Understanding these differences helps in designing effective database systems that can enforce complex rules and maintain consistency across both data and structure.

#### Common mistakes
include confusing the types of events that each trigger responds to, and not recognizing the importance of DML triggers for real-time data management and DDL triggers for long-term schema maintenance.

Code Blocks
SQL
-- Example of a DML Trigger
CREATE TRIGGER emp_update_log
AFTER UPDATE ON employees
FOR EACH ROW
BEGIN
    INSERT INTO audit_log (employee_id, action, timestamp)
    VALUES (OLD.employee_id, 'UPDATE', NOW());
END;

-- Example of a DDL Trigger
DELIMITER //
CREATE TRIGGER schema_backup
BEFORE DROP TABLE ON employees
FOR EACH ROW
BEGIN
    CREATE DATABASE backup_db;
    -- Perform backup operations here
END//
DELIMITER ;
Self-Check

Correctness and completeness confirmed.

