---
type: "practice"
difficulty: "Intermediate"
---

# What is a package in Oracle PL/SQL and what is the difference between the package specification and body?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ PL/SQL Overview](<../../../../TOOLS/Oracle/Oracle_Platform/PLSQL_Overview.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Oracle PL/SQL, a package is a fundamental unit of organization that allows you to encapsulate related PL/SQL procedures, functions, variables, constants, exceptions, and types within a single entity. The package consists of two parts: the specification (spec) and the body.

#### Triage

The question asks about the concept of a package in Oracle PL/SQL and differentiates between its specification and body. This is probing your understanding of encapsulation, organization, and the separation of concerns in PL/SQL development. You need to explain how each part functions separately but as a cohesive unit.

#### Assumptions
We assume you are familiar with basic PL/SQL programming concepts.
The package serves both as a security mechanism (by limiting access) and an optimization strategy by precompiling the code.
Step-by-Step Solution

Package Specification:

A package specification defines what is available to other parts of your application.
It includes declarations for:
Public types, constants, variables, and cursors.
Public procedures and functions with their parameters and return types.
The spec does not contain implementation details; it only provides a blueprint.

Package Body:

A package body contains the implementations of all the procedures, functions, etc., declared in the specification.
It includes detailed code for executing the functionality defined in the spec.
In some cases, you might declare private types, constants, variables here as well to keep implementation details hidden from other parts of your application.

Key Differences:

The package body is required to implement the functionality declared in the specification.
The separation ensures that only what is intended for public use is accessible outside the package.
It helps maintain a clean interface between different modules of an application, enhancing readability and ease of maintenance.

#### Closing Summary

In summary, a package in Oracle PL/SQL combines the specification with the body to encapsulate related PL/SQL entities. The spec defines what you can access externally, while the body contains the detailed implementation details. This separation promotes better organization, security, and reusability.

#### Common Mistakes/Follow-Ups
A common mistake is writing all code in a single file without separating declarations from implementations.
Another pitfall is not properly declaring private items that should remain hidden within the package.

#### Follow-ups
might include discussing how to handle public and private types, managing versioning, or explaining best practices for using packages.

**Code:**

```

-- Package Specification CREATE OR REPLACE PACKAGE my_package IS TYPE my_type IS RECORD (id NUMBER, name VARCHAR2(50));

PROCEDURE process_data(p_id IN NUMBER);

FUNCTION get_name(p_id IN NUMBER) RETURN VARCHAR2; END my_package; /

CODE
Copy

```plsql
-- Package Body
CREATE OR REPLACE PACKAGE BODY my_package IS

   -- Implementation of the procedures and functions declared in the spec
   PROCEDURE process_data(p_id IN NUMBER) IS
      v_rec my_type;
   BEGIN
      -- Logic to fetch data and update v_rec
   END process_data;

   FUNCTION get_name(p_id IN NUMBER) RETURN VARCHAR2 IS
      v_result VARCHAR2(50);
   BEGIN
      -- Fetch logic here
      RETURN v_result;
   END get_name;
END my_package;
/
```

**Flow:**

1. Package Specification
2. Declare Public Items
3. Define Interfaces
4. Implement with Details

