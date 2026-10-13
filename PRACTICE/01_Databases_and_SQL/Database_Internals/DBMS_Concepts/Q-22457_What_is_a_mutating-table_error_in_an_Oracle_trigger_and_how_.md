---
type: "practice"
difficulty: "Expert"
---

# What is a mutating-table error in an Oracle trigger and how do compound triggers or deferred logic get around it?

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ Lock Granularity](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Lock_Granularity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Mutating-table errors occur in database triggers when they try to access tables that are being modified by other statements within the same transaction, leading to potential inconsistencies. Compound and deferred triggers help mitigate these issues by allowing more granular control over when and how trigger actions are executed.

#### Triage

The question asks about two key concepts: mutating-table errors and how compound or deferred logic can address them in Oracle triggers. The core of the problem is understanding when and why mutating-table errors occur, and how advanced trigger mechanisms prevent these issues.

#### First-Principles Solution
Mutating-Table Error Overview

In an Oracle database, a mutating table error occurs when you attempt to query or update a table within a DML (Data Manipulation Language) statement that is modifying the same table. This can lead to logical inconsistencies because the data in the table might change between different parts of the transaction.

Compound Triggers and Deferred Logic

To avoid these errors, Oracle provides compound triggers and deferred logic features:

Compound Triggers: These allow you to group multiple SQL operations (like BEFORE, AFTER, etc.) into a single trigger body. This can reduce complexity by consolidating different parts of the transaction.

Deferred Logic: By deferring certain actions until later in the transaction, these mechanisms ensure that all changes are committed or rolled back together, preventing inconsistent states.

#### Example Scenario
s

#### Cons
ider a scenario where you need to maintain an audit log every time rows are inserted into a table:

Without proper handling: If multiple rows are inserted and the trigger tries to update another table based on the current state of the original table, it might encounter a mutating-table error.
With compound triggers or deferred logic: The action can be handled in such a way that it doesn’t rely on the current state of the table during row-level operations.
Full Solution
Mutating-Table Error Example

Let’s consider an example where we have two tables, employees and employee_audit, and we want to maintain an audit log every time employees are inserted or updated:

SQL
-- Without handling correctly
CREATE OR REPLACE TRIGGER emp_audit_trigger
BEFORE INSERT OR UPDATE ON employees
FOR EACH ROW
BEGIN
    IF INSERTING THEN
        INSERT INTO employee_audit (emp_id, action) VALUES (:NEW.emp_id, 'INSERT');
    ELSIF UPDATING THEN
        INSERT INTO employee_audit (emp_id, action) VALUES (:OLD.emp_id, 'UPDATE');
    END IF;
END;

This trigger will work fine in many cases, but if you perform a bulk insert or update with multiple rows, it might encounter a mutating-table error.

Using Compound Triggers

Now let’s implement this using a compound trigger:

SQL
CREATE OR REPLACE TRIGGER emp_audit_trigger
COMPOUND TRIGGER
    CURSOR c IS SELECT emp_id FROM employees FOR UPDATE OF emp_id;

BEGIN
    -- BEFORE EACH ROW processing (optional, can be used for validation or setup)

    FOR rec IN c LOOP
        IF INSERTING THEN
            INSERT INTO employee_audit (emp_id, action) VALUES (rec.emp_id, 'INSERT');
        ELSIF UPDATING THEN
            INSERT INTO employee_audit (emp_id, action) VALUES (rec.emp_id, 'UPDATE');
        END IF;
    END LOOP;

    -- AFTER EACH ROW processing (optional, can be used for cleanup or post-processing)
END emp_audit_trigger;

In this example, the FOR EACH ROW and AFTER EACH ROW sections allow you to process each row independently without relying on the current state of the table.

Deferred Logic Example

For deferred logic, consider using a single BEFORE STATEMENT section:

SQL
CREATE OR REPLACE TRIGGER emp_audit_trigger
BEFORE INSERT OR UPDATE ON employees
DECLARE
    CURSOR c IS SELECT emp_id FROM employees FOR UPDATE OF emp_id;
BEGIN
    OPEN c;

    LOOP
        FETCH c INTO v_emp_id;
        EXIT WHEN c%NOTFOUND;

        IF INSERTING THEN
            INSERT INTO employee_audit (emp_id, action) VALUES (v_emp_id, 'INSERT');
        ELSIF UPDATING THEN
            INSERT INTO employee_audit (emp_id, action) VALUES (v_emp_id, 'UPDATE');
        END IF;
    END LOOP;

    CLOSE c;
END emp_audit_trigger;

Here, all audit records are prepared in the BEFORE section and committed together at the end of the transaction.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Mutating-table errors can be avoided by using compound triggers or deferred logic. Compound triggers allow you to manage multiple rows more effectively within a single trigger body, while deferred logic ensures that actions are taken after all row-level changes have been committed or rolled back together.

#### Common mistakes
include:

Not considering the transaction context when writing triggers.
Attempting to modify a table directly in an AFTER EACH ROW section of a non-compound trigger.
Failing to handle bulk operations correctly, leading to potential logical inconsistencies.

#### Follow-ups
might involve discussing performance considerations, transaction management strategies, or specific use cases where one approach is more suitable than another.

