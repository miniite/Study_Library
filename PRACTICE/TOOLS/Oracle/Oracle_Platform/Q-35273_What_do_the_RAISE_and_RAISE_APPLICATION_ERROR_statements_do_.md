---
type: "practice"
difficulty: "Intermediate"
---

# What do the RAISE and RAISE_APPLICATION_ERROR statements do in Oracle PL/SQL?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ PL/SQL Overview](<../../../../TOOLS/Oracle/Oracle_Platform/PLSQL_Overview.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

RAISE and RAISE_APPLICATION_ERROR are both used to handle exceptions in Oracle PL/SQL, but they have different levels of granularity and use cases. In essence, RAISE is a general mechanism for raising exceptions, while RAISE_APPLICATION_ERROR allows you to define custom error messages with specific error codes.

#### Triage

Question: What do the RAISE and RAISE_APPLICATION_ERROR statements do in Oracle PL/SQL?
What’s being probed: Understanding of exception handling mechanisms in PL/SQL. Ability to differentiate between RAISE and RAISE_APPLICATION_ERROR, their syntax, and practical usage.

#### Assumptions
The reader has a basic understanding of PL/SQL exceptions.

Step-by-step solution:

RAISE Statement:

Functionality: RAISE is used to raise an exception within the body of a PL/SQL block. It can be used with predefined exceptions or custom ones.
Syntax:
SQL
RAISE [EXCEPTION_NAME];

#### Example
Usage:
SQL
DECLARE
  v_divisor NUMBER := 0;
BEGIN
  IF v_divisor = 0 THEN
    RAISE ZERO_DIVIDE;
  END IF;
EXCEPTION
  WHEN ZERO_DIVIDE THEN
    DBMS_OUTPUT.PUT_LINE('Cannot divide by zero!');
END;
Common Use Cases: Raising exceptions for specific conditions where predefined ones are not sufficient.

RAISE_APPLICATION_ERROR Statement:

Functionality: RAISE_APPLICATION_ERROR is used to raise a custom PL/SQL exception with an error number and message.
Syntax:
SQL
RAISE_APPLICATION_ERROR(error_number, error_message);

#### Example
Usage:
SQL
DECLARE
  v_divisor NUMBER := 0;
BEGIN
  IF v_divisor = 0 THEN
    RAISE_APPLICATION_ERROR(-20100, 'Cannot divide by zero!');
  END IF;
EXCEPTION
  WHEN -20100 THEN
    DBMS_OUTPUT.PUT_LINE('Custom error: Cannot divide by zero!');
END;
Common Use Cases: Defining and using custom error conditions that are not covered by predefined exceptions.

#### Closing Summary
Both RAISE and RAISE_APPLICATION_ERROR are crucial for robust exception handling in PL/SQL. While RAISE is more general, RAISE_APPLICATION_ERROR allows you to define custom error codes and messages, making your code more explicit and maintainable.

#### Common Mistakes/Follow-Ups

Failing to Use RAISE_APPLICATION_ERROR Correctly: Not providing a meaningful error number and message can make debugging difficult.
Overusing RAISE: Excessive use of RAISE without proper context or predefined exception handling can clutter the code.
Ignoring Exceptions: Ensuring that all exceptions are properly handled, either by catching them in the block or re-raising with RAISE.

**Code:**

```sql
-- Example RAISE Statement
DECLARE
  v_divisor NUMBER := 0;
BEGIN
  IF v_divisor = 0 THEN
    RAISE ZERO_DIVIDE;
  END IF;
EXCEPTION
  WHEN ZERO_DIVIDE THEN
    DBMS_OUTPUT.PUT_LINE('Cannot divide by zero!');
END;

-- Example RAISE_APPLICATION_ERROR Statement
DECLARE
  v_divisor NUMBER := 0;
BEGIN
  IF v_divisor = 0 THEN
    RAISE_APPLICATION_ERROR(-20100, 'Cannot divide by zero!');
  END IF;
EXCEPTION
  WHEN -20100 THEN
    DBMS_OUTPUT.PUT_LINE('Custom error: Cannot divide by zero!');
END;
```

