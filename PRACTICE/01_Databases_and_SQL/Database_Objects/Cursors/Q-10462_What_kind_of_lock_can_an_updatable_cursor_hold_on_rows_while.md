---
type: "practice"
difficulty: "Deep"
---

# What kind of lock can an updatable cursor hold on rows while it is open?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Ref Cursor Consistency and Limits](<../../../../01_Databases_and_SQL/Database_Objects/Cursors/Ref_Cursor_Consistency_and_Limits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When discussing cursors in a database context, we're dealing with a way to fetch and process rows from a result set one at a time. An "updatable cursor" is specifically designed not just to read data but also to modify it. The type of lock an updatable cursor can hold on rows while it is open depends on the specific implementation of the database system, its isolation levels, and the operations being performed.

Essentially, an updatable cursor will attempt to lock certain rows in the table to ensure that changes made via the cursor are isolated from other transactions. Let's dive into the details step-by-step.

**Comparison:**

| Cursor State | Lock Type    | Description                          |
|--------------|--------------|--------------------------------------|
| OPEN         | RowLockRead  | Reading a row                         |
| OPEN         | RowLockWrite | Modifying a row                       |

**Code:**

```sql
-- Example of using a cursor in SQL (PostgreSQL)
DECLARE emp_cursor CURSOR FOR SELECT * FROM employees;
FETCH NEXT FROM emp_cursor INTO @emp_id, @name, @salary; -- Fetching data
UPDATE employees SET salary = salary + 1000 WHERE id = @emp_id; -- Modifying data
CLOSE emp_cursor;
```

**Example input / output:**

```text
Input:
DECLARE emp_cursor CURSOR FOR SELECT * FROM employees;
FETCH NEXT FROM emp_cursor INTO @emp_id, @name, @salary;
UPDATE employees SET salary = salary + 1000 WHERE id = @emp_id;
CLOSE emp_cursor;

Output:
BEGIN (transaction starts)
-- Cursor is declared and opened on the employees table
Cursor state: OPEN, no rows fetched yet

FETCH NEXT FROM emp_cursor INTO @emp_id, @name, @salary; -- Fetches first row
Cursor state: Open, row 1 is locked for reading (RowLockRead)

UPDATE employees SET salary = salary + 1000 WHERE id = @emp_id; -- Modifies the fetched row
Cursor state: Open, row 1 is locked for writing (RowLockWrite)
```

**Data:**

```json
{
  "employees": [
    {"id": 1, "name": "John", "salary": 5000},
    {"id": 2, "name": "Jane", "salary": 6000}
  ]
}
```

**Flow:**

1. Row Lock for Reading
2. Row Lock for Writing
3. Cursor Declaration andOpening
4. Fetch Next
5. Row fetched
6. Update Operation
7. Row modified
8. CLOSE Cursor

