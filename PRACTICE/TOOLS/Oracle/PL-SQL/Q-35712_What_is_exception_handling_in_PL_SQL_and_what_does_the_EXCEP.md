---
type: "practice"
difficulty: "Basic"
---

# What is exception handling in PL/SQL and what does the EXCEPTION section do?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Exception Handling](<../../../../TOOLS/Oracle/PL-SQL/Exception_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Exception handling in PL/SQL is a crucial feature that allows developers to manage errors and unexpected conditions within database procedures. The EXCEPTION section of a PL/SQL block serves as a catch-all area where you can define how the program should respond when an error occurs.

Let's break down what exception handling does:

Identifying Exceptions: In PL/SQL, exceptions are predefined by Oracle (e.g., NO_DATA_FOUND,  TOO_MANY_ROWS, ZERO_DIVIDE). You can also declare custom exceptions.

Handling Exceptions: The EXCEPTION section allows you to write code that will execute if any of these exceptions occur during the execution of a PL/SQL block.

Exception Blocks: Within the EXCEPTION section, multiple exception blocks are used, each containing specific handling logic for different types of exceptions.

Here’s an example to illustrate this:

SQL
DECLARE
  v_age NUMBER := 12;
BEGIN
  IF v_age < 0 THEN
    RAISE_APPLICATION_ERROR(-20500, 'Age cannot be negative.');
  ELSIF v_age > 100 THEN
    RAISE_APPLICATION_ERROR(-20501, 'Age out of range.');
  END IF;

  -- Normal PL/SQL code

EXCEPTION
  WHEN NO_DATA_FOUND THEN
    DBMS_OUTPUT.PUT_LINE('No data found for the given condition.');
  WHEN OTHERS THEN
    DBMS_OUTPUT.PUT_LINE(SQLERRM);
END;

In this example, if v_age is negative or out of range, an appropriate error message will be raised and caught by the respective exception handlers. If any other unexpected error occurs, it will be handled by the OTHERS block.

**Code:**

```sql
DECLARE
  v_age NUMBER := 12;
BEGIN
  IF v_age < 0 THEN
    RAISE_APPLICATION_ERROR(-20500, 'Age cannot be negative.');
  ELSIF v_age > 100 THEN
    RAISE_APPLICATION_ERROR(-20501, 'Age out of range.');
  END IF;

  -- Normal PL/SQL code

EXCEPTION
  WHEN NO_DATA_FOUND THEN
    DBMS_OUTPUT.PUT_LINE('No data found for the given condition.');
  WHEN OTHERS THEN
    DBMS_OUTPUT.PUT_LINE(SQLERRM);
END;
```

