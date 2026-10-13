---
type: "practice"
difficulty: "Intermediate"
---

# How would you loop through query results and print each row using an explicit cursor?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ PL-SQL in Production](<../../../../TOOLS/Oracle/PL-SQL/PL-SQL_in_Production.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle this question, we need to understand how to use PL/SQL cursors effectively to iterate over query results in Oracle SQL. An explicit cursor allows us to declare a cursor, open it with a SELECT statement, fetch rows one at a time from the result set into variables, and process each row until all rows are fetched.

The core idea is that we'll create an explicit cursor to select data, use loops to process each row, and print or manipulate the data as needed. Let's walk through this step-by-step.

**Code:**

```
PLSQL
Copy
-- Declare a cursor for selecting data from a table

DECLARE
    -- Define variables to hold column values fetched from the cursor
    CURSOR emp_cursor IS
        SELECT employee_id, first_name, last_name FROM employees;

    -- Declare record type for fetching rows into
    rec_emp emp_cursor%ROWTYPE;
BEGIN
    -- Open the cursor
    OPEN emp_cursor;

    -- Loop through each row returned by the cursor
    LOOP
        FETCH emp_cursor INTO rec_emp;
        EXIT WHEN emp_cursor%NOTFOUND;  -- Exit loop when no more records are found

        -- Print or process the fetched row here (e.g., print employee details)
        DBMS_OUTPUT.PUT_LINE(rec_emp.employee_id || ', ' ||
                             rec_emp.first_name || ' ' || rec_emp.last_name);
    END LOOP;

    -- Close the cursor after processing all rows
    CLOSE emp_cursor;
END;
```

**Example input / output:**

```

None provided, but here's an example of how it works:

Input: The employees table with columns employee_id, first_name, and last_name.
Output:
CODE
Copy
101, John Smith
102, Jane Doe
...
```

