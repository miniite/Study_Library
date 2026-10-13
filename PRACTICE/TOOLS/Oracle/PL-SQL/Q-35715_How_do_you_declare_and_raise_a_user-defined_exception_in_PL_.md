---
type: "practice"
difficulty: "Intermediate"
---

# How do you declare and raise a user-defined exception in PL/SQL?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Exception Handling](<../../../../TOOLS/Oracle/PL-SQL/Exception_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this explanation, I'll cover how to declare and raise a user-defined exception in PL/SQL. This is crucial for handling specific errors that are relevant to your application logic rather than generic SQL exceptions.

#### Triaging the Question

The core of the question revolves around understanding when and why you might want to define and use custom exceptions. You need to understand how these exceptions differ from built-in ones, how they can be declared in a PL/SQL block, and how to raise them appropriately. The interviewer is probing your knowledge of exception handling in PL/SQL, your ability to handle specific error conditions, and your understanding of best practices.

#### First-Principles Solution

In PL/SQL, you can declare user-defined exceptions using the EXCEPTION keyword within a block. This allows you to create meaningful names for common errors or application-specific issues that aren't covered by built-in exceptions. The syntax involves declaring an exception and then raising it when necessary.

Declaring User-Defined Exceptions

To declare a user-defined exception, use:

SQL
DECLARE
   <exception_name> EXCEPTION;
BEGIN
   -- Your code here
EXCEPTION
   WHEN <exception_name> THEN
      -- Handle the exception
END;

Here’s an example to illustrate:

SQL
DECLARE
   too_many_records FOUNDATION_EXCEPTION;  -- Custom exception declaration
BEGIN
   -- Some logic that might trigger the exception
   IF (SELECT COUNT(*) FROM employees > 10) THEN
      RAISE too_many_records;  -- Raising a user-defined exception
   END IF;
EXCEPTION
   WHEN too_many_records THEN
      DBMS_OUTPUT.PUT_LINE('Too many records found.');
END;
Raising Exceptions

To raise an exception, use the RAISE statement followed by the name of your custom exception. The example above shows how to raise a user-defined exception named too_many_records.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, declaring and raising user-defined exceptions in PL/SQL is key for robust error handling that aligns with application logic. Always ensure meaningful names are used and appropriate error messages are provided.

#### Common mistakes
include:

Not defining the exception in a DECLARE block.
Forgetting to handle (raise) the custom exception within the EXCEPTION section.
Using built-in exceptions where user-defined ones would be more precise and useful.

#### Follow-ups
could involve discussing best practices for naming conventions, logging mechanisms, and how these exceptions integrate with transaction management.

**Code:**

```sql
DECLARE
   too_many_records FOUNDATION_EXCEPTION;  -- Custom exception declaration
BEGIN
   -- Some logic that might trigger the exception
   IF (SELECT COUNT(*) FROM employees > 10) THEN
      RAISE too_many_records;  -- Raising a user-defined exception
   END IF;
EXCEPTION
   WHEN too_many_records THEN
      DBMS_OUTPUT.PUT_LINE('Too many records found.');
END;
```

