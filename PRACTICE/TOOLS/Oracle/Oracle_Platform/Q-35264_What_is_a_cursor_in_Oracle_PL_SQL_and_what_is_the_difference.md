---
type: "practice"
difficulty: "Intermediate"
---

# What is a cursor in Oracle PL/SQL and what is the difference between an implicit and explicit cursor?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ PL/SQL Overview](<../../../../TOOLS/Oracle/Oracle_Platform/PLSQL_Overview.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A cursor in Oracle PL/SQL is a mechanism to control access to rows in a database table or view. There are two main types of cursors: implicit cursors, which are automatically created by the Oracle engine when a query is executed, and explicit cursors, which are manually declared by the developer for more complex operations.

To triage this question, we need to understand that both types serve different purposes and have distinct characteristics. Implicit cursors are simpler to use but less efficient and harder to control compared to explicit cursors, which offer more flexibility in handling complex query logic. Let's explore each type from first principles.

**Comparison:**

| Employee ID | First Name | Last Name |
|-------------|------------|-----------|
| 123         | John       | Doe       |

**Code:**

```

-- Example of an implicit cursor (SELECT statement) SELECT * FROM employees WHERE employee_id = 123;

-- Example of an explicit cursor declaration and usage DECLARE CURSOR emp_cursor IS SELECT * FROM employees; BEGIN OPEN emp_cursor; -- Open the cursor to start processing rows FETCH emp_cursor INTO v_employee_record; -- Fetch a row from the cursor into a variable WHILE emp_cursor%FOUND LOOP DBMS_OUTPUT.PUT_LINE(v_employee_record.first_name || ' ' || v_employee_record.last_name); FETCH emp_cursor INTO v_employee_record; END LOOP; CLOSE emp_cursor; -- Close the cursor to free resources END;

CODE
Copy

```sql
-- Example data schema for employees table
CREATE TABLE employees (
   employee_id NUMBER,
   first_name VARCHAR2(50),
   last_name VARCHAR2(50)
);
```

**Example input / output:**

```

Let's walk through an input-output scenario for both implicit and explicit cursors.

Implicit Cursor Example (SELECT Statement):

SQL
Copy
-- Input: SELECT * FROM employees WHERE employee_id = 123;
-- Output:
+-----------+------------+-------------+
| EMPLOYEE_ID | FIRST_NAME | LAST_NAME   |
+-----------+------------+-------------+
|         123 | John       | Doe         |
+-----------+------------+-------------+

Explicit Cursor Example (Fetch Loop):

SQL
Copy
DECLARE
   CURSOR emp_cursor IS
      SELECT * FROM employees;
   v_employee_record employees%ROWTYPE;
BEGIN
   OPEN emp_cursor; -- Open the cursor to start processing rows
   FETCH emp_cursor INTO v_employee_record; -- Fetch a row from the cursor into a variable

   WHILE emp_cursor%FOUND LOOP
      DBMS_OUTPUT.PUT_LINE(v_employee_record.first_name || ' ' || v_employee_record.last_name);
      FETCH emp_cursor INTO v_employee_record;
   END LOOP;

   CLOSE emp_cursor; -- Close the cursor to free resources
END;

Output:

CODE
Copy
John Doe
```

**Data:**

```sql
-- Sample data insertion into employees table
INSERT INTO employees (employee_id, first_name, last_name) VALUES (123, 'John', 'Doe');
COMMIT;
```

**Flow:**

1. Yes
2. No
3. Implicit Cursor Example
4. Is query simple?
5. SELECT * FROM employeesWHERE employee_id = 123;
6. Explicit Cursor Example
7. DECLARE CURSOR emp_cursorIS SELECT * FROM employees;
8. OPEN emp_cursor;
9. FETCH emp_cursor INTOv_employee_record; WHILEemp_cursor%FOUND LOOP ...END LOOP; CLOSE emp_cursor;

