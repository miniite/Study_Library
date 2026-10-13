---
type: "practice"
difficulty: "Intermediate"
---

# How do you use ROW_NUMBER() to pick one representative row per duplicate group before deleting the rest?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ ROW_NUMBER RANK and DENSE_RANK](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/ROW_NUMBER_RANK_and_DENSE_RANK.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

ROW_NUMBER() is a powerful window function that allows us to assign a unique sequential integer to rows within a partition of a result set, which can be used to pick one representative row from duplicate groups before deleting the rest. To achieve this, we first identify the duplicate groups using some common criteria and then use ROW_NUMBER() to rank these duplicates based on certain conditions.

#### Triaging the Question

The interviewer is likely probing your understanding of:

Window Functions: Specifically, how ROW_NUMBER() works.
Duplicate Detection and Deletion: How you can identify and handle duplicate rows in a database.
SQL Syntax and Logic: Ensuring you have a solid grasp of SQL queries and common operations.
Step-by-Step Solution

Let's assume we have a table called employees with the following schema:

SQL
CREATE TABLE employees (
    id INT,
    name VARCHAR(100),
    department VARCHAR(50),
    salary DECIMAL(10, 2)
);

We want to pick one representative employee per department (based on some criteria) and delete all others.

Step 1: Identify the Duplicate Groups

First, we need to identify employees who are duplicates within each department. For simplicity, let's consider an "employee" as a duplicate if they have the same name in the same department.

SQL
WITH duplicate_groups AS (
    SELECT
        name,
        department
    FROM
        employees
    GROUP BY
        name, department
    HAVING COUNT(*) > 1
)
Step 2: Assign ROW_NUMBER() to Each Duplicate Group

Next, we use ROW_NUMBER() within each group to assign a unique number. We can rank the rows based on some criteria like salary or hire date.

SQL
WITH duplicate_groups AS (
    SELECT
        name,
        department
    FROM
        employees
    GROUP BY
        name, department
    HAVING COUNT(*) > 1
),
ranked_duplicates AS (
    SELECT
        e.*,
        ROW_NUMBER() OVER (PARTITION BY e.name, e.department ORDER BY e.salary DESC) as rn
    FROM
        employees e
    INNER JOIN
        duplicate_groups dg ON e.name = dg.name AND e.department = dg.department
)
Step 3: Delete All Rows Except the Representative One

Finally, we delete all rows except for the one with rn = 1.

SQL
DELETE FROM employees e
WHERE (e.name, e.department) IN (
    SELECT
        name,
        department
    FROM
        ranked_duplicates rd
    WHERE
        rn > 1
);

#### Closing Summary
and Common Mistakes/Follow-Ups

This solution leverages window functions to manage duplicate rows by assigning a row number based on certain criteria. The key is correctly identifying the duplicates and using ROW_NUMBER() appropriately. Some common mistakes include:

Failing to properly partition the data.
Not considering edge cases like ties in the ranking criterion (e.g., multiple employees with the same salary).

For follow-ups, you might be asked about performance considerations or alternative methods.

Triaging for Deeper Insights
Performance: Ensure that the query is optimized, especially if dealing with large datasets. Indexes on name and department could help.
Alternative Methods: Explore other window functions like RANK(), DENSE_RANK(), etc., to see how they might change the outcome.

**Comparison:**

```
Start
Identify Duplicate Groups
RANK each duplicate group
Delete non-representativerows
End
```

**Code:**

```sql
WITH duplicate_groups AS (
    SELECT
        name,
        department
    FROM
        employees
    GROUP BY
        name, department
    HAVING COUNT(*) > 1
),
ranked_duplicates AS (
    SELECT
        e.*,
        ROW_NUMBER() OVER (PARTITION BY e.name, e.department ORDER BY e.salary DESC) as rn
    FROM
        employees e
    INNER JOIN
        duplicate_groups dg ON e.name = dg.name AND e.department = dg.department
)
DELETE FROM employees e
WHERE (e.name, e.department) IN (
    SELECT
        name,
        department
    FROM
        ranked_duplicates rd
    WHERE
        rn > 1;
);
```

**Example input / output:**

```

WITH duplicate_groups AS ( SELECT name, department FROM employees GROUP BY name, department HAVING COUNT() > 1 ), ranked_duplicates AS ( SELECT e., ROW_NUMBER() OVER (PARTITION BY e.name, e.department ORDER BY e.salary DESC) as rn FROM employees e INNER JOIN duplicate_groups dg ON e.name = dg.name AND e.department = dg.department ) SELECT * FROM ranked_duplicates;

CODE
Copy

Output:
```sql
+----+---------+-----------+----------+
| id | name    | department| salary   |
+----+---------+-----------+----------+
| 2  | Alice   | Finance   | 70000.00 |
| 5  | David   | IT        | 85000.00 |
| 3  | Bob     | HR        | 58000.00 |
+----+---------+-----------+----------+
SQL
Copy
DELETE FROM employees e
WHERE (e.name, e.department) IN (
    SELECT
        name,
        department
    FROM
        ranked_duplicates rd
    WHERE
        rn > 1;
);

SELECT * FROM employees;

Output:

SQL
Copy
+----+---------+-----------+----------+
| id | name    | department| salary   |
+----+---------+-----------+----------+
| 2  | Alice   | Finance   | 70000.00 |
| 4  | Charlie | IT        | 90000.00 |
+----+---------+-----------+----------+
```

**Data:**

```sql
CREATE TABLE employees (
    id INT,
    name VARCHAR(100),
    department VARCHAR(50),
    salary DECIMAL(10, 2)
);

INSERT INTO employees (id, name, department, salary) VALUES
(1, 'John', 'HR', 60000),
(2, 'Alice', 'Finance', 70000),
(3, 'Bob', 'HR', 58000),
(4, 'Charlie', 'IT', 90000),
(5, 'David', 'IT', 85000);
```

**Flow:**

1. Create Table
2. INSERT data
3. WITH Duplicate Groups AS ...
4. SELECT * FROMranked_duplicates
5. DELETE non-representativerows
6. Select remaining rows

