---
type: "practice"
difficulty: "Basic"
---

# What is a PL/SQL package and what are its two parts?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ PL-SQL in Production](<../../../../TOOLS/Oracle/PL-SQL/PL-SQL_in_Production.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A PL/SQL package in Oracle databases is like organizing your room into boxes—each box contains similar items, making it easier to manage and find what you need. It consists of two main parts: a specification (the "box label") that describes the public elements available for use, and the body (the contents inside) where the actual implementation happens. This separation allows for better structure, reusability, and security.

The package specification is like putting a label on your box; it defines what the package offers to the outside world—procedures, functions, types, constants, and variables that are accessible from other parts of the database. The body contains the actual implementation of these elements, hidden behind the closed lid of the box. This separation helps in maintaining code integrity and security.

Let's break this down step by step:

Define the Package Specification: Here you declare what is available to be used outside the package.

SQL
CREATE OR REPLACE PACKAGE my_package IS
    PROCEDURE do_something;
    FUNCTION get_value RETURN NUMBER;
END my_package;

Implement the Package Body: In this part, you write the actual code for the procedures and functions declared in the specification.

SQL
CREATE OR REPLACE PACKAGE BODY my_package IS
    PROCEDURE do_something IS
        -- implementation details
    BEGIN
        -- procedure logic
    END do_something;

    FUNCTION get_value RETURN NUMBER IS
        -- function logic
    BEGIN
        RETURN 10;
    END get_value;
END my_package;

This approach ensures that the internal workings of a package remain hidden, enhancing security and maintainability. For example, if get_value returns a complex calculation, having it in the body keeps your external code cleaner.

#### Common mistakes
to avoid include:

Not defining all public elements in the specification.
Overloading the body with too much logic that should be kept separate for clarity and reusability.
Failing to properly document the package's purpose and usage.

**Code:**

```sql
-- Example of a PL/SQL Package Specification
CREATE OR REPLACE PACKAGE my_package IS
    PROCEDURE do_something;
    FUNCTION get_value RETURN NUMBER;
END my_package;

-- Example of a PL/SQL Package Body
CREATE OR REPLACE PACKAGE BODY my_package IS
    PROCEDURE do_something IS
        -- implementation details
    BEGIN
        -- procedure logic
    END do_something;

    FUNCTION get_value RETURN NUMBER IS
        -- function logic
    BEGIN
        RETURN 10;
    END get_value;
END my_package;
```

