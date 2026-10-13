---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between a predefined exception and a user-defined exception?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Exception Handling](<../../../../TOOLS/Oracle/PL-SQL/Exception_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In PL/SQL, both predefined exceptions and user-defined exceptions serve to handle errors or exceptional conditions during database operations, but they differ in their nature and how they are declared and handled.

TL;DR:
Predefined Exceptions are built into the PL/SQL language and cover common error scenarios.
User-Defined Exceptions allow developers to create custom exception handlers for specific situations not covered by predefined ones.

#### Triage
What's Being Probed

The interviewer is likely probing your understanding of exception handling in PL/SQL, including the ability to distinguish between different types of exceptions and when you might use each. You'll need to explain both concepts clearly and provide examples where appropriate.

#### First-Principles Solution
Predefined Exceptions:

Predefined exceptions are those that come with the PL/SQL language and cover common error scenarios such as NO_DATA_FOUND, too_many_rows, or ZERO_DIVIDE. These exceptions do not need to be declared by the developer. They are implicitly recognized by the compiler.

Here's an example of handling a predefined exception:

PLSQL
BEGIN
    -- Some code that might raise NO_DATA_FOUND
    IF NOT CURSOR_NAME%FOUND THEN
        RAISE_APPLICATION_ERROR(-20101, 'No data found in cursor');
    END IF;
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('No data was found as expected.');
END;
User-Defined Exceptions:

User-defined exceptions are created by the developer to handle specific error conditions that might occur during program execution. These can be declared using the DECLARE section of a PL/SQL block and should have an exception handler in the EXCEPTION section.

Here's how you would declare and use a user-defined exception:

PLSQL
DECLARE
    NOT_FOUND_EXCEPTION EXCEPTION;
BEGIN
    -- Some code that might raise NOT_FOUND_EXCEPTION
    IF NOT CURSOR_NAME%FOUND THEN
        RAISE NOT_FOUND_EXCEPTION;
    END IF;
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('No data was found as expected.');
    WHEN NOT_FOUND_EXCEPTION THEN
        DBMS_OUTPUT.PUT_LINE('The requested record was not found.');
END;

#### Closing Summary
+ Common Mistakes/Follow-Ups

To summarize, predefined exceptions are useful for handling common error conditions without needing additional code, whereas user-defined exceptions allow you to handle specific scenarios that might arise in your application. You should use both as needed, but always consider whether the exception is commonly known or if it represents a unique situation.

#### Common Mistakes
Forgetting to declare and raise user-defined exceptions properly.
Not handling all relevant predefined exceptions.
Overusing user-defined exceptions where predefined ones suffice.

#### Follow-up
questions might include asking how you would implement error logging, what trade-offs exist between using different types of exceptions, or in which scenarios one type is preferred over the other.

**Code:**

```
PLSQL
Copy
-- Example of handling a predefined exception
BEGIN
    -- Some code that might raise NO_DATA_FOUND
    IF NOT CURSOR_NAME%FOUND THEN
        RAISE_APPLICATION_ERROR(-20101, 'No data found in cursor');
    END IF;
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('No data was found as expected.');
END;

-- Example of using a user-defined exception
DECLARE
    NOT_FOUND_EXCEPTION EXCEPTION;
BEGIN
    -- Some code that might raise NOT_FOUND_EXCEPTION
    IF NOT CURSOR_NAME%FOUND THEN
        RAISE NOT_FOUND_EXCEPTION;
    END IF;
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('No data was found as expected.');
    WHEN NOT_FOUND_EXCEPTION THEN
        DBMS_OUTPUT.PUT_LINE('The requested record was not found.');
END;
```

