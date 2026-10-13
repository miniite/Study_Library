---
type: "practice"
difficulty: "Deep"
---

# What is the difference between a strong REF CURSOR and a weak REF CURSOR in PL/SQL?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Cursor Basics and Ref Cursors](<../../../../01_Databases_and_SQL/Database_Objects/Cursors/Cursor_Basics_and_Ref_Cursors.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In PL/SQL, cursors are used to manage rows returned by a query or other data manipulation operations. The two main types of REF cursors are strong and weak. A strong REF cursor holds the actual reference to the data in the database, while a weak REF cursor does not hold any such reference, making it less memory-intensive but also more limited in functionality.

To fully understand this concept, let's break down each type:

Strong REF Cursor: These cursors are typically used when you need to work with specific data that is stored in the database and might be updated independently of the cursor itself. They allow for direct manipulation of the underlying data.

Weak REF Cursor: These cursors are useful when you don't require a strong reference to the data, which can help save memory but limits your ability to directly modify the data.

The primary difference between these two types lies in how they handle memory and data persistence. Now let's explore this step by step:

Step-by-Step Explanation
Strong REF Cursor
Memory Usage: A strong REF cursor holds a reference to an object in the database, which means it takes up more memory since it essentially copies the state of the row.
Data Persistence: Because it maintains a direct reference, changes made through the cursor can be committed and flushed back to the database, allowing for updates to the actual data.
Flexibility: Strong cursors offer more flexibility in handling complex operations like updates or deletes.
Weak REF Cursor
Memory Usage: A weak REF cursor does not hold a direct reference to the data, making it less memory-intensive.
Data Persistence: Changes made through a weak cursor are not automatically committed back to the database. The changes are only visible within the scope of the transaction where they were made and do not affect the actual row in the database unless explicitly committed.
Limitations: Weak cursors are primarily used for read-only operations or when you need to minimize memory usage.

#### Example Scenario

Suppose we have a table employees with columns id, name, and salary. We want to update an employee's salary based on their ID.

SQL
-- Using a strong REF cursor
DECLARE
  emp_cur SYS_REFCURSOR;
  v_salary NUMBER;
BEGIN
  OPEN emp_cur FOR 'SELECT salary FROM employees WHERE id = :1', 101;

  FETCH emp_cur INTO v_salary;

  -- Modify the data directly through the cursor (strong reference)
  UPDATE employees SET salary = v_salary * 1.1 WHERE CURRENT OF emp_cur;

  CLOSE emp_cur;
END;

-- Using a weak REF cursor
DECLARE
  weak_emp CURSOR FOR 'SELECT salary FROM employees WHERE id = :1' USING BULK COLLECT INTO TABLE v_salaries, 101; -- Not valid syntax, for illustrative purposes only.
BEGIN
  OPEN weak_emp;

  -- Changes made here are not committed to the database until explicitly done
  UPDATE employees SET salary = v_salary * 1.2 WHERE id IN (SELECT id FROM TABLE(v_salaries));

  CLOSE weak_emp;
END;

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, choose a strong REF cursor when you need direct manipulation of the data in the database or when updates are required. Use a weak REF cursor to minimize memory usage for read-only operations where changes are not critical.

#### Common mistakes
include:

Failing to commit changes made through a strong REF cursor.
Assuming that a weak REF cursor automatically commits changes back to the database, leading to unintentional data loss.

For follow-ups, consider scenarios where you might use both types together in a single application or explore more advanced usage cases like bulk operations and transaction management.

