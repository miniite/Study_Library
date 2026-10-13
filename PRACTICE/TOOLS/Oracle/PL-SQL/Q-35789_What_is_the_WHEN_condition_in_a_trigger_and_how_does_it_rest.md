---
type: "practice"
difficulty: "Intermediate"
---

# What is the WHEN condition in a trigger and how does it restrict trigger execution?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Trigger Types](<../../../../TOOLS/Oracle/PL-SQL/Trigger_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When discussing triggers in PL/SQL, one of the crucial components is the WHEN condition. This condition helps control when the trigger should be executed based on certain criteria. Understanding how WHEN conditions work is essential for writing efficient and targeted triggers.

To start, let's break down what a WHEN condition is and how it affects trigger execution. A WHEN clause in PL/SQL allows you to specify conditions that must be true before the associated trigger body is executed. These conditions can include various predicates like equality checks, logical operators, and more.

For example, consider a scenario where you want to ensure that a new record being inserted into a table meets certain criteria, such as having a specific value in one of its columns. You could use a WHEN condition to restrict the trigger's execution based on these conditions.

Here’s an example to illustrate:

SQL
CREATE OR REPLACE TRIGGER before_insert_check
BEFORE INSERT ON employees
FOR EACH ROW
WHEN (NEW.dept_id = 10)
BEGIN
    -- Trigger body
END;

In this case, the trigger before_insert_check will only execute if a new record being inserted into the employees table has a dept_id of 10. If the value of NEW.dept_id is not 10, the trigger will not fire.

#### Triage
What and Why

The question essentially asks about the role and usage of the WHEN condition in PL/SQL triggers. The interviewer might be probing for your understanding of:

The syntax and placement of the WHEN clause.
How to use different conditions within a WHEN clause.
The impact on performance when using or not using WHEN.

#### First Principles Solution

Syntax: A WHEN condition is used in PL/SQL triggers by placing it after the trigger keyword (e.g., BEFORE, AFTER) and before the table name. It can be used with various types of conditions, such as equality checks, inequalities, logical operators, or even more complex expressions.

Usage: The primary purpose of a WHEN condition is to control the execution flow based on specific criteria. This helps in creating efficient triggers by avoiding unnecessary executions when certain conditions are not met.

#### Example
Let's consider another example where you want to restrict an update operation if the salary being updated exceeds a certain threshold:

SQL
CREATE OR REPLACE TRIGGER before_update_salary_check
BEFORE UPDATE OF salary ON employees
FOR EACH ROW
WHEN (NEW.salary > 10000)
BEGIN
    -- Handle error or log message
END;

In this example, the trigger before_update_salary_check will only execute if the new value of salary being updated exceeds 10,000. If the update is for a salary less than or equal to 10,000, the trigger will not run.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, the WHEN condition in PL/SQL triggers allows you to control when the associated block of code should be executed based on specific conditions. This feature helps in writing more efficient and targeted triggers by avoiding unnecessary executions under certain circumstances.

#### Common mistakes
include:

Not using WHEN clauses where they would significantly improve trigger performance.
Incorrectly specifying conditions, leading to unintended behavior or failures.

For follow-ups, the interviewer might ask about:

The impact of WHEN conditions on transactional integrity and performance.
How WHEN conditions interact with other PL/SQL constructs like IF-THEN-ELSE.

**Code:**

```

CREATE OR REPLACE TRIGGER before_insert_check BEFORE INSERT ON employees FOR EACH ROW WHEN (NEW.dept_id = 10) BEGIN -- Trigger body END;

CODE
Copy

```sql
CREATE OR REPLACE TRIGGER before_update_salary_check
BEFORE UPDATE OF salary ON employees
FOR EACH ROW
WHEN (NEW.salary > 10000)
BEGIN
    -- Handle error or log message
END;
```

