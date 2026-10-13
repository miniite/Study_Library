---
type: "practice"
difficulty: "Intermediate"
---

# What exception is raised when a SELECT INTO returns no rows and how do you handle it?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Exception Handling](<../../../../TOOLS/Oracle/PL-SQL/Exception_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When using SELECT INTO in PL/SQL, if the query does not return any rows, then an implicit NO_DATA_FOUND exception is raised. This means that the statement couldn't retrieve any data into the variables you specified. Handling this correctly ensures your application can gracefully manage situations where expected records might be missing.

To handle this, we need to explicitly catch and manage the NO_DATA_FOUND exception in our PL/SQL code. Let’s break down how we would implement this:

#### Triage
Question: What is the exception raised when a SELECT INTO statement does not return any rows?
Probing Points: Understanding of implicit exceptions, handling specific SQL-related errors.

#### Assumptions
Familiarity with basic PL/SQL syntax and exception handling.

#### First Principles Solution
Identify the Error Condition: When using SELECT INTO, if no row is found matching the query criteria, it implicitly raises a NO_DATA_FOUND exception.
Handle the Exception Explicitly: Use an EXCEPTION block to catch this specific error and take appropriate actions (e.g., logging, setting default values).

#### Example
Code
SQL
-- Example PL/SQL block that handles NO_DATA_FOUND when using SELECT INTO
DECLARE
    v_employee_id NUMBER;
    v_employee_name VARCHAR2(100);
BEGIN
    -- Attempt to select data into variables
    BEGIN
        SELECT employee_id, first_name || ' ' || last_name INTO v_employee_id, v_employee_name
        FROM employees
        WHERE employee_id = 9999;  -- Assuming there is no such record

    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            DBMS_OUTPUT.PUT_LINE('No data found for the given ID.');
            -- Optionally set default values or take other actions here.

    END;

    -- Continue with your application logic
END;
/

#### Closing Summary
and Common Mistakes/Follow-Ups

In this example, we've used a BEGIN...EXCEPTION block to handle the potential NO_DATA_FOUND exception. It's important to log or alert when such an exception occurs so you can understand why no data was found.

#### Common Mistakes

Not handling NO_DATA_FOUND, leading to runtime errors.
Incorrectly catching other exceptions instead of NO_DATA_FOUND.

#### Follow-Ups

How would you ensure the same logic works in a stored procedure or function?
What if there could be multiple records returned? Would we use a different exception?

