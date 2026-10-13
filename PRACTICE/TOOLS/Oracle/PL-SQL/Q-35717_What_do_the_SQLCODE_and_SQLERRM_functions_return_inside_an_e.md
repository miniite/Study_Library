---
type: "practice"
difficulty: "Intermediate"
---

# What do the SQLCODE and SQLERRM functions return inside an exception handler?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Exception Handling](<../../../../TOOLS/Oracle/PL-SQL/Exception_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with exceptions in PL/SQL, SQLCODE and SQLERRM are two important functions that provide valuable information about the nature of the error encountered. Here’s a brief overview:

Summary: In an exception block within PL/SQL, SQLCODE returns the numeric error code associated with the SQL error, while SQLERRM provides the corresponding error message.

TL;DR: Use these functions to handle and log errors effectively.

Now let's dive into how they work and their implications:

#### Triage

The question is asking about the return values of two key functions (SQLCODE and SQLERRM) used in PL/SQL exception handling. The interviewer is probing your understanding of error handling mechanisms, the importance of numeric codes versus messages, and how to handle exceptions efficiently.

#### Assumptions
You are familiar with basic PL/SQL syntax.
Errors in SQL can be categorized by both numbers and descriptive messages.
Exception handlers should provide meaningful feedback for debugging and maintenance purposes.

#### Solution
from First Principles

Understanding SQLCODE:

SQLCODE returns an integer value that corresponds to the specific error encountered during a PL/SQL block or function execution.
The range of values can vary, but typically, negative numbers indicate errors (e.g., -20594 for a no_data_found exception).
Positive numbers are generally reserved for warnings and informational messages.

Understanding SQLERRM:

SQLERRM returns the error message associated with the SQLCODE.
It provides a textual description of what went wrong, making it easier to understand the problem without manually looking up codes in documentation or books.
However, its behavior can vary depending on whether an explicit exception was raised or if the system automatically handles and logs errors.

#### Example
Usage:

PLSQL
DECLARE
  v_dept_id NUMBER := 999;
BEGIN
  SELECT COUNT(*) INTO v_dept_id FROM departments WHERE department_id = v_dept_id;
  -- This will raise a NO_DATA_FOUND exception.

EXCEPTION
  WHEN NO_DATA_FOUND THEN
    DBMS_OUTPUT.PUT_LINE('SQLCODE: ' || SQLCODE);  -- Outputs the error code, e.g., -1403
    DBMS_OUTPUT.PUT_LINE('SQLERRM: ' || SQLERRM);  -- Outputs the error message, e.g., "NO_DATA_FOUND - no data found"
END;

#### Closing Summary
and Common Mistakes/Follow-Ups
Closing: Always use SQLCODE and SQLERRM together to log exceptions effectively. This combination helps in diagnosing issues quickly.

#### Common Mistakes
Relying solely on SQLCODE without the context provided by SQLERRM.
Not handling exceptions gracefully, leading to application crashes or silent failures.

For follow-ups, you might be asked about how different types of errors are handled (e.g., syntax errors vs. logic errors) or about best practices for logging and troubleshooting in a production environment.

**Comparison:**

| Error Code (SQLCODE) | Error Message (SQLERRM)          |
|----------------------|----------------------------------|
| -20594               | NO_DATA_FOUND - no data found    |
| -1722                | ORA-01722: invalid number         |

**Code:**

```
PLSQL
Copy
DECLARE
  v_dept_id NUMBER := 999;
BEGIN
  SELECT COUNT(*) INTO v_dept_id FROM departments WHERE department_id = v_dept_id;
  -- This will raise a NO_DATA_FOUND exception.
EXCEPTION
  WHEN NO_DATA_FOUND THEN
    DBMS_OUTPUT.PUT_LINE('SQLCODE: ' || SQLCODE);  -- Outputs the error code, e.g., -1403
    DBMS_OUTPUT.PUT_LINE('SQLERRM: ' || SQLERRM);  -- Outputs the error message, e.g., "NO_DATA_FOUND - no data found"
END;
```

**Example input / output:**

| Input | Output |
|-------|--------|
| No data in the `departments` table for `department_id = 999`. | SQLCODE: -1403, SQLERRM: NO_DATA_FOUND - no data found. |

**Data:**

```plaintext
Sample data:
- departments table with department_id as primary key.
```

**Flow:**

1. Begin PL/SQL Block
2. Execute SQL Statement
3. No Data Found
4. Output SQLCODE andSQLERRM
5. Data Found
6. End PL/SQL Block

