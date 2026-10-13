---
type: "practice"
difficulty: "Intermediate"
---

# How do you pass parameters by position versus by name when calling a PL/SQL procedure?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Functions in SQL vs PL/SQL](<../../../../TOOLS/Oracle/PL-SQL/Functions_in_SQL_vs_PLSQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When calling a PL/SQL procedure, you can pass parameters either by position or by name. Passing by position means using the order of the parameters as defined in the procedure signature, while passing by name allows you to explicitly specify which parameter corresponds to each value being passed. Both methods have their use cases and advantages.

The key difference lies in how you specify the parameters when calling a procedure:

Passing Parameters by Position:

You pass values based on the position of the parameters as defined in the procedure signature.
If the procedure has three parameters, you would provide three arguments in order: procedure_name(param1_value, param2_value, param3_value).

Passing Parameters by Name:

You explicitly specify which parameter each value corresponds to using named parameters: procedure_name(param1 => param1_value, param2 => param2_value, param3 => param3_value).
This method can be more readable and easier to understand, especially with many or complex parameters.

#### Triaging the Question

The interviewer is probing your understanding of PL/SQL syntax and parameter passing mechanisms. They may also want to know about potential trade-offs between using position-based and name-based parameters in different contexts.

#### First-Principles Solution
Passing Parameters by Position

Here’s an example procedure:

SQL
CREATE OR REPLACE PROCEDURE update_employee (
    emp_id IN NUMBER,
    emp_name IN VARCHAR2,
    emp_salary IN NUMBER
) AS
BEGIN
    -- Procedure logic here
END;

To call this procedure using position-based parameters, you would do:

SQL
EXECUTE update_employee(101, 'John Doe', 50000);
Passing Parameters by Name

You can also call the same procedure using named parameters:

SQL
EXECUTE update_employee(emp_id => 101, emp_name => 'John Doe', emp_salary => 50000);

This approach is particularly useful when the order of parameters might be confusing or you want to make the code more readable.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, passing PL/SQL procedure parameters by position or name depends on the context. Position-based parameters are straightforward but can lead to errors if the order changes. Name-based parameters enhance readability and maintainability, especially in complex procedures with many parameters.

#### Common mistakes
include:

Forgetting to change the parameter order when modifying a procedure.
Overusing named parameters unnecessarily for simple procedures.
Common Follow-Ups

The interviewer might ask about performance differences between both methods, or how you would choose one over the other based on specific use cases. They might also probe deeper into handling optional and default parameters in PL/SQL.

**Code:**

```sql
-- Example of a procedure with position-based call
CREATE OR REPLACE PROCEDURE update_employee (
    emp_id IN NUMBER,
    emp_name IN VARCHAR2,
    emp_salary IN NUMBER
) AS
BEGIN
    -- Procedure logic here
END;

-- Position-based call
EXECUTE update_employee(101, 'John Doe', 50000);

-- Name-based call
EXECUTE update_employee(emp_id => 101, emp_name => 'John Doe', emp_salary => 50000);
```

