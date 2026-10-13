---
type: "practice"
difficulty: "Deep"
---

# What is a compound trigger and what problem does it help solve?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Trigger Types](<../../../../TOOLS/Oracle/PL-SQL/Trigger_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A compound trigger in PL/SQL allows you to define multiple points of execution within a single trigger, which can greatly enhance the flexibility and functionality of database operations. It helps in managing complex business logic that spans across different DML (Data Manipulation Language) statements such as INSERT, UPDATE, DELETE, or even SELECT.

#### Triage

The question is asking about compound triggers in PL/SQL and what problems they solve. The interviewer is likely probing your understanding of:

What a trigger is.
The differences between simple and compound triggers.
The benefits of using compound triggers for managing complex business logic.

#### First-Principles Solution

Let's start with the basics: A trigger in PL/SQL is a database object that automatically executes a block of code (a stored procedure) when certain events occur, such as inserting, updating, or deleting data. Triggers can be either simple or compound.

A compound trigger is a special kind of PL/SQL trigger that allows the developer to define multiple actions at different points during the execution of DML statements. These points include:

BEFORE STATEMENT and AFTER STATEMENT: Executed before and after the DML statement.
BEFORE EACH ROW and AFTER EACH ROW: Executed for each row affected by the DML statement.

#### Example Scenario

Imagine you have a table employees with columns such as id, name, salary, and date_of_birth. You want to enforce business rules:

Ensure that no employee's salary is less than $20,000.
Automatically calculate the age of an employee based on their date of birth.

Using a simple trigger for each rule would result in redundant code execution. However, with a compound trigger, you can handle both these requirements at different points:

SQL
CREATE OR REPLACE TRIGGER emp_salary_and_age_trigger
FOR INSERT OR UPDATE OF salary, date_of_birth ON employees
COMPOUND TRIGGER

  -- Before any rows are processed
  BEFORE STATEMENT IS
    BEGIN
      IF UPDATING('salary') THEN
        FOR rec IN (SELECT * FROM employees WHERE salary < 20000) LOOP
          DBMS_OUTPUT.PUT_LINE('Employee with ID ' || rec.id || ' has a salary below the minimum.');
        END LOOP;
      END IF;
    END BEFORE STATEMENT;

  -- After each row is processed
  AFTER EACH ROW IS
    BEGIN
      -- Calculate age based on date of birth
      :new.age := TO_NUMBER(TO_CHAR(SYSDATE - :new.date_of_birth, 'J')) / 365.25;
    END AFTER EACH ROW;

END emp_salary_and_age_trigger;

#### Closing Summary
and Common Mistakes/Follow-Ups

Compound triggers are incredibly powerful for managing complex business logic that spans multiple DML operations. However, they can also lead to performance issues if not used judiciously:

Redundant Code: Ensure you avoid duplicating code across different points within the trigger.
Complexity: Compound triggers can become quite intricate and harder to maintain. A good practice is to keep each section (BEFORE STATEMENT, AFTER EACH ROW) as simple as possible.

Common follow-up questions might include:

How would you handle DML operations on multiple tables?
What are the performance implications of using compound triggers in large-scale applications?

**Comparison:**

```
Column	Data Type
id	INT
name	VARCHAR(100)
salary	DECIMAL
date_of_birth	DATE
age	INT
```

**Code:**

```sql
CREATE OR REPLACE TRIGGER emp_salary_and_age_trigger
FOR INSERT OR UPDATE OF salary, date_of_birth ON employees
COMPOUND TRIGGER

  -- Before any rows are processed
  BEFORE STATEMENT IS
    BEGIN
      IF UPDATING('salary') THEN
        FOR rec IN (SELECT * FROM employees WHERE salary < 20000) LOOP
          DBMS_OUTPUT.PUT_LINE('Employee with ID ' || rec.id || ' has a salary below the minimum.');
        END LOOP;
      END IF;
    END BEFORE STATEMENT;

  -- After each row is processed
  AFTER EACH ROW IS
    BEGIN
      -- Calculate age based on date of birth
      :new.age := TO_NUMBER(TO_CHAR(SYSDATE - :new.date_of_birth, 'J')) / 365.25;
    END AFTER EACH ROW;

END emp_salary_and_age_trigger;
```

**Example input / output:**

```

No specific input-output examples are necessary for this explanation, but here's a conceptual example:

Input: Inserting an employee with a salary below $20,000.
Output: A message indicating the violation of the business rule.
PLAINTEXT
Copy
Employee with ID 1 has a salary below the minimum.
```

**Data:**

```sql
CREATE TABLE employees (
  id INT,
  name VARCHAR(100),
  salary DECIMAL,
  date_of_birth DATE,
  age INT
);
```

**Flow:**

1. INSERT OR UPDATE OFsalary, date_of_birth
2. Yes
3. Trigger Activation
4. Is DML Statement?
5. BEFORE STATEMENT
6. Check Salary Rule Violations
7. Output Message
8. AFTER EACH ROW
9. Calculate Age

