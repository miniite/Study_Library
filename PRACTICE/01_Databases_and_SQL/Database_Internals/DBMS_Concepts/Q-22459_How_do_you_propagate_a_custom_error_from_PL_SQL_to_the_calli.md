---
type: "practice"
difficulty: "Intermediate"
---

# How do you propagate a custom error from PL/SQL to the calling application using RAISE_APPLICATION_ERROR and named exceptions?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Dynamic SQL](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Dynamic_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When propagating a custom error from PL/SQL back to an application, using RAISE_APPLICATION_ERROR is a common practice in Oracle databases. This function allows you to raise an exception with a user-defined message and severity level, which can then be caught and handled by the calling application. In this answer, I'll explain how to use RAISE_APPLICATION_ERROR, discuss its syntax, provide a step-by-step example, and highlight potential pitfalls.

#### Triaging the Question

The question is essentially asking you to describe and demonstrate the process of raising a custom error in PL/SQL that can be caught and handled by an application. The key aspects being probed are your understanding of:

RAISE_APPLICATION_ERROR usage.
Exception handling mechanisms in PL/SQL.
Error management best practices.
Step-by-Step Solution
Define the Error: Use a negative number (between -20999 and -2099) as the error number, which Oracle reserves for user-defined errors.
Set Severity Level: The severity level is an integer between 0 and 10, where 1-6 are warnings and 7-10 are critical errors.
Raise the Error: Use RAISE_APPLICATION_ERROR to raise the custom error with a descriptive message.

#### Example
Implementation

Let's say you need to validate a customer ID in your PL/SQL stored procedure before performing some operation. If the customer ID does not exist, you want to return an error to the application.

PLSQL
DECLARE
  v_customer_id NUMBER := 123456; -- Example customer ID
BEGIN
  IF NOT EXISTS (SELECT 1 FROM customers WHERE id = v_customer_id) THEN
    RAISE_APPLICATION_ERROR(-20001, 'Customer ID not found: ' || v_customer_id);
  END IF;

  -- Continue with further operations...
END;
/

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, RAISE_APPLICATION_ERROR is a powerful tool for creating custom error conditions in PL/SQL. It allows you to define specific errors that can be caught by the application layer, providing more context and control over what happens when an unexpected condition arises.

#### Common mistakes
include:

Using non-reserved error numbers (which will not work).
Forgetting to handle the exception on the calling side.
Not including descriptive messages in the error.

#### Follow-up
questions might include how you would manage multiple custom errors, integrate with logging systems, or handle different severity levels differently.

**Code:**

```
PLSQL
Copy
DECLARE
  v_customer_id NUMBER := 123456; -- Example customer ID
BEGIN
  IF NOT EXISTS (SELECT 1 FROM customers WHERE id = v_customer_id) THEN
    RAISE_APPLICATION_ERROR(-20001, 'Customer ID not found: ' || v_customer_id);
  END IF;

  -- Continue with further operations...
END;
/
```

