---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between a local variable and a global variable in a PL/SQL package?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Functions in SQL vs PL/SQL](<../../../../TOOLS/Oracle/PL-SQL/Functions_in_SQL_vs_PLSQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Local and global variables in PL/SQL packages serve different purposes and have distinct scopes, which makes them useful in various contexts but can also lead to confusion if not understood properly.

#### Triage

The core of this question is about understanding the scope and visibility of variables within a PL/SQL package. A local variable is declared within a subprogram (like a procedure or function) and is only accessible within that specific block of code. On the other hand, a global variable is declared at the package level and can be accessed throughout the entire package.

#### Solution
Local Variables
Definition: Local variables are declared within a PL/SQL subprogram like a procedure or function.
Scope: These variables are only accessible inside the block where they are declared, which could be a single function, procedure, or even an anonymous block of code (like a DECLARE section in a script).
Visibility: Local variables cannot be accessed from outside their declaring subprogram.
Global Variables
Definition: Global variables are declared at the package level.
Scope: These variables can be accessed by any procedure, function, or anonymous block within the same package.
Visibility: Global variables provide a way to share data between different parts of your PL/SQL code that belong to the same package.

#### Example

Here’s an example to illustrate:

PLSQL
CREATE OR REPLACE PACKAGE test_package IS
   g_global_var NUMBER;  -- A global variable declared at the package level
END;
/

-- Inside a procedure in the package
CREATE OR REPLACE PACKAGE BODY test_package IS
   PROCEDURE local_procedure IS
      l_local_var NUMBER;  -- Local to this procedure
   BEGIN
      DBMS_OUTPUT.PUT_LINE('Local Var: ' || l_local_var);  -- This will work within the block
   END;

   PROCEDURE global_procedure IS
   BEGIN
      g_global_var := 10;  -- Accessing and modifying a global variable
      DBMS_OUTPUT.PUT_LINE('Global Var: ' || g_global_var);  -- Also works here
   END;
END;
/

#### Closing Summary

In summary, local variables are scoped to the block where they are declared (e.g., within a procedure), while global variables have broader access and can be used across multiple procedures in the same package. Understanding these differences is crucial for effective PL/SQL development.

#### Common Mistakes/Follow-Ups
Misunderstanding Scope: One common mistake is confusing local and global variables, thinking that they are accessible where they should not be.
Incorrect Initialization: Forgetting to initialize a global variable properly can lead to unexpected behavior.
Accessing Non-Existent Global Variables: Trying to use a global variable from outside its package scope will result in an error.

Would you like me to elaborate on any of these points further?

**Code:**

```
PLSQL
Copy
CREATE OR REPLACE PACKAGE test_package IS
   g_global_var NUMBER;  -- A global variable declared at the package level
END;
/

-- Inside a procedure in the package
CREATE OR REPLACE PACKAGE BODY test_package IS
   PROCEDURE local_procedure IS
      l_local_var NUMBER;  -- Local to this procedure
   BEGIN
      DBMS_OUTPUT.PUT_LINE('Local Var: ' || l_local_var);  -- This will work within the block
   END;

   PROCEDURE global_procedure IS
   BEGIN
      g_global_var := 10;  -- Accessing and modifying a global variable
      DBMS_OUTPUT.PUT_LINE('Global Var: ' || g_global_var);  -- Also works here
   END;
END;
/
```

**Example input / output:**

```text
Local Var: (variable not initialized, so it will be NULL or default value)
Global Var: 10
```

