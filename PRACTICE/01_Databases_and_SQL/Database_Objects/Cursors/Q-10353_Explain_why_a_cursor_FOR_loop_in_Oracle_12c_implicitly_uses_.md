---
type: "practice"
difficulty: "Deep"
---

# Explain why a cursor FOR loop in Oracle 12c+ implicitly uses an array fetch of 100 rows and when you would override that behavior.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Cursor Basics and Ref Cursors](<../../../../01_Databases_and_SQL/Database_Objects/Cursors/Cursor_Basics_and_Ref_Cursors.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Oracle 12c and later, cursors use implicit array fetching by default, where each FETCH operation retrieves up to 100 rows at a time. This is done to improve performance and reduce network traffic. However, in some scenarios, you might want to override this behavior for more granular control over the data retrieval process.

**Code:**

```sql
-- Example of an Oracle cursor with implicit array fetching
DECLARE
  CURSOR emp_cursor IS
    SELECT employee_id, first_name, last_name FROM employees;
BEGIN
  FOR rec IN emp_cursor LOOP
    DBMS_OUTPUT.PUT_LINE(rec.first_name || ' ' || rec.last_name);
  END LOOP;
END;
/
```

**Example input / output:**

```text
-- Example input for fetching rows from the cursor
emp_cursor: SELECT employee_id, first_name, last_name FROM employees;

-- Output showing how Oracle fetches rows in chunks of 100
1. First FETCH operation retrieves row 1 to 100.
2. Second FETCH operation retrieves row 101 to 200.
3. This continues until all rows are processed or the cursor is closed.
```

**Flow:**

1. Yes
2. No
3. Continue
4. End
5. Cursor Declaration
6. Is Implicit Array Fetch Enabled?
7. Fetch 100 rows at a time
8. Fetch specified number ofrows
9. Process Rows
10. Process Rows
11. Continue Fetching or CloseCursor?
12. End Cursor Processing

