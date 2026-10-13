---
type: "practice"
difficulty: "Intermediate"
---

# What are the cursor attributes %FOUND, %NOTFOUND, %ROWCOUNT, and %ISOPEN in Oracle?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Oracle Aggregate Functions](<../../../../TOOLS/Oracle/Oracle_Platform/Oracle_Aggregate_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Cursor attributes like %FOUND, %NOTFOUND, %ROWCOUNT, and %ISOPEN in Oracle Database are essential for understanding the state of cursor operations and help developers manage loops and handle query results effectively.

To start, let's define what these attributes do:

%FOUND: This attribute returns a boolean value indicating whether the last FETCH operation was successful or not.
%NOTFOUND: This is essentially the inverse of %FOUND; it returns true if the last FETCH operation did not find any more rows to fetch.
%ROWCOUNT: It stores the number of rows fetched by the cursor since its last open. Useful for knowing how many records were processed in a loop or a query execution.
%ISOPEN: Indicates whether the cursor is currently open and ready for operations like FETCH or CLOSE.

Now, let's look at each attribute step-by-step from first principles:

Step 1: Understanding Cursor Operations

First, consider that cursors are used to fetch data from SQL queries in a loop. For example:

SQL
DECLARE
    CURSOR emp_cursor IS SELECT * FROM employees;
BEGIN
    FOR emp_rec IN emp_cursor LOOP
        -- Do something with each row
    END LOOP;
END;
Step 2: Exploring %FOUND and %NOTFOUND

When a FETCH operation is performed, it attempts to get the next row from the cursor. If successful, %FOUND will be set to true; otherwise, if no more rows are available, %NOTFOUND will be true.

SQL
DECLARE
    CURSOR emp_cursor IS SELECT * FROM employees;
BEGIN
    FOR emp_rec IN emp_cursor LOOP
        IF %FOUND THEN
            -- Process the current row
        ELSE
            -- No more rows found
        END IF;
    END LOOP;
END;
Step 3: %ROWCOUNT for Tracking Rows Fetched

This attribute is particularly useful when you need to keep track of how many rows have been processed. For example, in a loop that processes data:

SQL
DECLARE
    CURSOR emp_cursor IS SELECT * FROM employees;
BEGIN
    FOR emp_rec IN emp_cursor LOOP
        -- Process the current row
        IF %ROWCOUNT MOD 100 = 0 THEN
            DBMS_OUTPUT.PUT_LINE('Processed ' || TO_CHAR(%ROWCOUNT) || ' rows so far');
        END IF;
    END LOOP;
END;
Step 4: %ISOPEN for Cursor Management

This attribute is crucial when you need to ensure that a cursor is open before performing any operations. For instance, if you want to check the state of a cursor after closing it:

SQL
DECLARE
    CURSOR emp_cursor IS SELECT * FROM employees;
BEGIN
    OPEN emp_cursor; -- Open the cursor
    IF %ISOPEN THEN
        FOR emp_rec IN emp_cursor LOOP
            -- Process each row
        END LOOP;
    ELSE
        DBMS_OUTPUT.PUT_LINE('Cursor is not open');
    END IF;

    CLOSE emp_cursor; -- Close the cursor

    -- Check if the cursor is still open after closing it
    IF %ISOPEN THEN
        DBMS_OUTPUT.PUT_LINE('Cursor should be closed, but it is still open');
    ELSE
        DBMS_OUTPUT.PUT_LINE('Cursor has been properly closed');
    END IF;
END;

#### Closing Summary

These cursor attributes provide developers with powerful tools to manage cursors and handle SQL operations more effectively. %FOUND and %NOTFOUND help in managing loops, while %ROWCOUNT is useful for tracking the number of rows processed. Finally, %ISOPEN ensures that you can safely perform operations on a cursor only when it is open.

#### Common Mistakes/Follow-Ups
Mistake: Ignoring %FOUND and %NOTFOUND in loops, leading to unexpected behavior or infinite loops.

#### Follow-Up
Discuss the performance implications of using these attributes versus other methods like COUNT(*).

**Code:**

```sql
DECLARE
    CURSOR emp_cursor IS SELECT * FROM employees;
BEGIN
    FOR emp_rec IN emp_cursor LOOP
        IF %FOUND THEN
            -- Process each row
        ELSE
            -- No more rows found
        END IF;
    END LOOP;
END;
```

