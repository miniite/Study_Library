---
type: "practice"
difficulty: "Deep"
---

# What is the difference between the PRAGMA EXCEPTION_INIT declaration and just checking SQLCODE after the fact?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Dynamic SQL](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Dynamic_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with a database, understanding how to handle exceptions correctly can significantly improve the robustness of your application. The PRAGMA EXCEPTION_INIT declaration in Oracle PL/SQL allows you to map a specific SQLCODE to an exception name, which is different from simply checking SQLCODE after an operation has failed. Let's explore this concept step by step.

#### Triage
Understanding the Question

The question asks about the difference between using PRAGMA EXCEPTION_INIT and just checking SQLCODE. We need to understand that:

SQLCODE provides a numeric code indicating the type of error.
PRAGMA EXCEPTION_INIT maps SQLCODEs to exception names, making your error handling more descriptive.
First Principles: Detailed Explanation
SQLCODE

The SQLCODE mechanism is straightforward but can be cumbersome. It returns a numeric value that corresponds to an SQL error. For example:

SQL
BEGIN
  -- Some operation that might fail
EXCEPTION
  WHEN OTHERS THEN
    DBMS_OUTPUT.PUT_LINE('Error occurred: SQLCODE ' || SQLCODE);
END;

This approach is effective but can lead to less descriptive messages and makes it harder to manage multiple error conditions.

PRAGMA EXCEPTION_INIT

PRAGMA EXCEPTION_INIT provides a more structured way of handling exceptions by associating SQLCODEs with named exceptions. This makes the code easier to read and maintain:

SQL
BEGIN
  -- Some operation that might fail

  -- Map specific SQLCODEs to named exceptions
  PRAGMA EXCEPTION_INIT(my_custom_exception, -20500);

EXCEPTION
  WHEN my_custom_exception THEN
    DBMS_OUTPUT.PUT_LINE('Error occurred: Custom exception');
  WHEN OTHERS THEN
    DBMS_OUTPUT.PUT_LINE('Error occurred: Default exception handling');
END;

Here, my_custom_exception is a user-defined exception that will be raised if the SQLCODE -20500 occurs. This makes your code more readable and maintainable.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, using PRAGMA EXCEPTION_INIT offers a cleaner and more maintainable way to handle exceptions compared to simply checking SQLCODE. It aligns with best practices in PL/SQL for error handling. However, it is essential to ensure that the mappings are correct to avoid runtime errors.

#### Common mistakes
include:

Mismatching SQLCODE values.
Overusing OTHERS exception handlers without proper mapping.
Not documenting the exceptions adequately.

#### Follow-up
questions might involve discussing scenarios where one approach would be preferred over the other or exploring additional PL/SQL features for error handling.

**Code:**

```sql
-- Example of PRAGMA EXCEPTION_INIT usage

BEGIN
  -- Some operation that might fail

  -- Map specific SQLCODEs to named exceptions
  PRAGMA EXCEPTION_INIT(my_custom_exception, -20500);

EXCEPTION
  WHEN my_custom_exception THEN
    DBMS_OUTPUT.PUT_LINE('Error occurred: Custom exception');
  WHEN OTHERS THEN
    DBMS_OUTPUT.PUT_LINE('Error occurred: Default exception handling');
END;
```

