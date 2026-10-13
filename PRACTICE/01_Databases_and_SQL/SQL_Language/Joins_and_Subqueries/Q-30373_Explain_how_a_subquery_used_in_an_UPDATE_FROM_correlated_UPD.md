---
type: "practice"
difficulty: "Deep"
---

# Explain how a subquery used in an UPDATE ... FROM / correlated UPDATE can update the wrong rows if it returns multiple matches, and how to make it deterministic.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Rewriting Subqueries as Joins](<../../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Rewriting_Subqueries_as_Joins.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When using UPDATE ... FROM or correlated updates with subqueries, there's a common pitfall where the subquery might return more than one row, leading to unintended data modifications. This is because the update operation can mistakenly apply changes across multiple rows that match the subquery condition. In this answer, I'll explain how this happens and provide strategies to ensure deterministic behavior.

#### Triage
What's Being Probed

The question revolves around understanding the intricacies of SQL UPDATE statements with subqueries, specifically focusing on the risk of updating too many rows or none at all due to ambiguous conditions. The interviewer is likely probing your knowledge of query semantics and the importance of ensuring correct updates in critical database operations.

#### Assumptions
You are familiar with basic SQL syntax.
You understand the concept of correlated queries and subqueries within UPDATE statements.
You know about JOIN operations and their usage in updating rows based on multiple tables.

#### First Principles Solution
Understanding the Problem: Ambiguous Updates

#### Cons
ider a scenario where you want to update a table, say Employees, by setting a column value based on a condition that involves another table, Departments. The subquery might look like this:

SQL
UPDATE Employees e SET e.salary = (SELECT d.salary_threshold FROM Departments d WHERE d.id = e.department_id)

Here, the subquery is correlated because it depends on the current row being updated. However, if there are multiple Departments with different salary_thresholds, and an employee's department can have multiple such departments (perhaps due to historical reasons or some ambiguity), this could lead to unintended updates.

#### Example Scenario

Imagine a simplified schema:

SQL
CREATE TABLE Employees (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    salary DECIMAL(10, 2),
    department_id INT
);

CREATE TABLE Departments (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    salary_threshold DECIMAL(10, 2)
);

Now, suppose Employees and Departments have the following data:

SQL
INSERT INTO Employees (id, name, salary, department_id) VALUES (1, 'Alice', 40000, 1), (2, 'Bob', 50000, 1);
INSERT INTO Departments (id, name, salary_threshold) VALUES (1, 'Sales', 55000), (2, 'Tech', 60000);

If you run the UPDATE statement:

SQL
UPDATE Employees e SET e.salary = (SELECT d.salary_threshold FROM Departments d WHERE d.id = e.department_id)

You might end up with both Alice and Bob having their salaries set to the same threshold of $55,000 because the correlated subquery returned multiple rows.

Deterministic Behavior: Using EXISTS

To avoid this issue, you can use an EXISTS clause to ensure that only a single row is updated when the condition holds. The revised query would look like:

SQL
UPDATE Employees e
SET e.salary = d.salary_threshold
FROM Departments d
WHERE d.id = e.department_id AND EXISTS (SELECT 1 FROM Departments WHERE id = e.department_id)

Here, EXISTS ensures that only rows where the condition holds are updated, preventing multiple rows from being affected.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, when using correlated subqueries in an UPDATE ... FROM statement, it's crucial to ensure that the subquery returns a single row. Using EXISTS can help enforce this requirement and make your updates deterministic. Always test your queries with small datasets or debug statements to catch such issues early.

#### Common mistakes
include:

Not understanding how correlated subqueries work.
Assuming that any match in the subquery will automatically lead to an update, leading to unintended results.
Failing to validate the logic through testing or simulation.

For follow-ups, you might be asked about performance implications of using EXISTS vs. subqueries, or scenarios where you would prefer one over the other based on specific use cases.

**Comparison:**

| id | name  | salary | department_id |
|----|-------|--------|--------------|
| 1  | Alice | 40000  | 1            |
| 2  | Bob   | 50000  | 1            |
| id | name  | salary_threshold |
|----|-------|------------------|
| 1  | Sales | 55000            |
| 2  | Tech  | 60000            |

**Code:**

```sql
-- Example schema
CREATE TABLE Employees (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    salary DECIMAL(10, 2),
    department_id INT
);

CREATE TABLE Departments (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    salary_threshold DECIMAL(10, 2)
);

-- Insert data
INSERT INTO Employees (id, name, salary, department_id) VALUES (1, 'Alice', 40000, 1), (2, 'Bob', 50000, 1);
INSERT INTO Departments (id, name, salary_threshold) VALUES (1, 'Sales', 55000), (2, 'Tech', 60000);

-- Ambiguous update
UPDATE Employees e SET e.salary = (SELECT d.salary_threshold FROM Departments d WHERE d.id = e.department_id);

-- Deterministic update using EXISTS
UPDATE Employees e
SET e.salary = d.salary_threshold
FROM Departments d
WHERE d.id = e.department_id AND EXISTS (SELECT 1 FROM Departments WHERE id = e.department_id);
```

**Example input / output:**

```text
-- Before and after states of Employees table

Before:
| id | name  | salary | department_id |
|----|-------|--------|--------------|
| 1  | Alice | 40000  | 1            |
| 2  | Bob   | 50000  | 1            |

Ambiguous update (incorrect):
| id | name  | salary |
|----|-------|--------|
| 1  | Alice | 55000  |
| 2  | Bob   | 55000  |

Deterministic update (correct):
| id | name  | salary |
|----|-------|--------|
| 1  | Alice | 55000  |
| 2  | Bob   | 50000  |
```

**Data:**

```sql
CREATE TABLE Employees (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    salary DECIMAL(10, 2),
    department_id INT
);

CREATE TABLE Departments (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    salary_threshold DECIMAL(10, 2)
);

INSERT INTO Employees (id, name, salary, department_id) VALUES (1, 'Alice', 40000, 1), (2, 'Bob', 50000, 1);
INSERT INTO Departments (id, name, salary_threshold) VALUES (1, 'Sales', 55000), (2, 'Tech', 60000);
```

**Flow:**

1. Start
2. Define the problem scenario
3. Correlated subquery returnsmultiple rows
4. Deterministic update usingEXISTS
5. End

