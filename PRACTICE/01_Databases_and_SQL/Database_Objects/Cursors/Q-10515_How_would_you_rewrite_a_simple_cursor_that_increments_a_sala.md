---
type: "practice"
difficulty: "Intermediate"
---

# How would you rewrite a simple cursor that increments a salary column into a single set-based UPDATE?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Cursors vs Set-based SQL](<../../../../01_Databases_and_SQL/Database_Objects/Cursors/Cursors_vs_Set-based_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question, we need to understand what a cursor does and then find an alternative approach using set-based operations in SQL. A cursor allows us to loop through rows one by one and perform operations on them, which can be less efficient than set-based operations that operate on the entire table at once.

#### Triaging the Question

The interviewer is likely probing your understanding of cursor versus set-based operations, performance considerations, and your ability to refactor code. We assume you have a basic knowledge of SQL and are familiar with cursors.

Step-by-Step Solution from First Principles

Let's say we have an employees table with columns id, name, and salary. The original cursor-based approach might look like this:

SQL
DECLARE @current_salary INT;
DECLARE employee_cursor CURSOR FOR SELECT salary FROM employees;

OPEN employee_cursor;
FETCH NEXT FROM employee_cursor INTO @current_salary;

WHILE @@FETCH_STATUS = 0
BEGIN
    -- Process each row
    UPDATE employees SET salary = @current_salary + 1 WHERE id = (SELECT id FROM employees WHERE salary = @current_salary);

    FETCH NEXT FROM employee_cursor INTO @current_salary;
END

CLOSE employee_cursor;
DEALLOCATE employee_cursor;

This approach fetches each salary, updates the corresponding row's salary by incrementing it, and then moves to the next salary.

However, this can be inefficient because for each iteration, we query the entire table again to find the matching id. Instead, a more efficient set-based approach would update all rows in one statement. Here’s how you could refactor it:

SQL
UPDATE employees
SET salary = salary + 1;

#### Closing Summary
and Common Mistakes/Follow-Ups

The refactored solution is much simpler and performs better because it avoids the overhead of repeatedly querying the table inside a loop. This approach ensures that each row's salary column is updated exactly once, which is faster and more efficient.

#### Common mistakes
include not recognizing when a cursor can be replaced with a set-based operation or failing to understand why set-based operations are generally preferred for performance reasons. Follow-ups might involve discussing scenarios where cursors are necessary (like complex logic per row) or exploring other optimization techniques like indexing.

