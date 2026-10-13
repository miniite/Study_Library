---
type: "practice"
difficulty: "Intermediate"
---

# How do you view compilation errors of a procedure using SHOW ERRORS or USER_ERRORS?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Exception Handling](<../../../../TOOLS/Oracle/PL-SQL/Exception_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how to view compilation errors of a procedure in PL/SQL, it's important to first grasp that these errors can occur during the creation or modification of PL/SQL code and are crucial for debugging. The SHOW ERRORS command allows us to inspect the details of such errors, while USER_ERRORS is a dynamic performance view (DPV) that stores information about compilation issues in our current schema.

#### Triage
Understanding the Question

The question is essentially asking how to effectively use PL/SQL commands (SHOW ERRORS, USER_ERRORS) for diagnosing and resolving compilation errors. The key point here is understanding the difference between these two commands, their syntax, and when each should be used.

#### Assumptions
We are using Oracle Database.
The user has the necessary privileges to create and alter PL/SQL procedures.
The procedure being created or modified is causing a compile-time error.

#### First-Principles Solution
Using SHOW ERRORS

The SHOW ERRORS command provides detailed information about the compilation errors. This command can be used in several ways:

Syntax for Creating a Procedure:

SQL
CREATE OR REPLACE PROCEDURE my_procedure IS
  -- procedure body
BEGIN
  -- code
END;
/

SHOW ERRORS

Output of SHOW ERRORS: The output will include the line number, severity level (e.g., note, warning, error), and a brief description of the error.

Using USER_ERRORS

The USER_ERRORS view is more detailed and can be queried directly to get information about all compilation errors in your schema. This is useful for managing multiple issues or when you need to review historical data.

Syntax:

SQL
SELECT * FROM USER_ERRORS;

Output of USER_ERRORS:

NAME: Name of the object with a problem.
LINE: Line number in the source code where the error occurred.
ERROR: Detailed description of the error.
SEVERITY: Level of the error (e.g., SUCCESS, ERROR, WARNING).

#### Closing Summary
and Common Mistakes

Both SHOW ERRORS and USER_ERRORS are powerful tools for debugging PL/SQL compilation errors. However, it's important to note that:

Common Mistake: Ignoring detailed line numbers provided by SHOW ERRORS. This can lead to missing the exact location of the error in your code.
Common Mistake: Overlooking historical data stored in USER_ERRORS, which might provide context for recurring issues.

#### Example
Worked Input->Output

Suppose we create a procedure and encounter an error:

SQL
CREATE OR REPLACE PROCEDURE test_procedure IS
  var1 NUMBER;
BEGIN
  SELECT COUNT(*) INTO var1 FROM non_existent_table; -- This will cause an error
END;
/
SHOW ERRORS;

-- Output might look like:
LINE/COL ERROR
---------- -----------------------------------------------------------------
6/34      PLS-00201: identifier 'NON_EXISTENT_TABLE' must be declared

By using SHOW ERRORS, we can quickly identify that the table name is misspelled.

Common Follow-Ups and Extensions
Understanding Error Severity: Different severity levels might indicate different issues, such as warnings or critical errors.
Using USER_ERRORS for Historical Data: If you need to refer back to previous compilation issues, USER_ERRORS can be very useful.
Automating Error Handling: For large projects, consider writing scripts to automate the retrieval and handling of compilation errors.

**Code:**

```sql
-- Example using SHOW ERRORS
CREATE OR REPLACE PROCEDURE test_procedure IS
  var1 NUMBER;
BEGIN
  SELECT COUNT(*) INTO var1 FROM non_existent_table; -- This will cause an error
END;
/
SHOW ERRORS;

-- Using USER_ERRORS for more detailed information
SELECT * FROM USER_ERRORS WHERE SEVERITY = 'ERROR';
```

**Example input / output:**

```text
-- Example input and output using SHOW ERRORS
CREATE OR REPLACE PROCEDURE test_procedure IS
  var1 NUMBER;
BEGIN
  SELECT COUNT(*) INTO var1 FROM non_existent_table; -- This will cause an error
END;
/
SHOW ERRORS;

-- Output:
LINE/COL ERROR
---------- -----------------------------------------------------------------
6/34      PLS-00201: identifier 'NON_EXISTENT_TABLE' must be declared

-- Example using USER_ERRORS for more detailed information
SELECT * FROM USER_ERRORS WHERE SEVERITY = 'ERROR';

-- Output might look like:
NAME          LINE ERROR                                               SEVERITY
------------- ---- -------------------------------------------------------------------- -------
TEST_PROCEDURE 6   PLS-00201: identifier 'NON_EXISTENT_TABLE' must be declared    ERROR
```

