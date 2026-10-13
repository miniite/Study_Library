---
type: "practice"
difficulty: "Intermediate"
---

# How would you list departments that have more than 10 employees using GROUP BY and HAVING?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Basic Query Practice](<../../../../TOOLS/MySQL/MySQL_Platform/Basic_Query_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to use SQL's GROUP BY clause along with the HAVING clause to filter groups of departments based on a condition. Specifically, we want to list all departments that have more than 10 employees.

#### Triaging the Question

The question is asking for a SQL query to retrieve department names where each department has more than 10 employees. The key idea here is using GROUP BY to aggregate data by department and then filtering these groups with HAVING based on the number of employees.

Step-by-Step Solution from First Principles

#### Assumptions
We have a table named employees which contains columns such as employee_id, name, and department_id.
The department_id column uniquely identifies each department.
Each employee is associated with one department.
SQL Query
SQL
SELECT department_id, COUNT(*) AS num_employees
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 10;

#### Closing Summary
and Common Mistakes/Follow-Ups

This query groups the employees by their department_id, counts the number of employees in each group, and then filters out those departments with fewer than or equal to 10 employees. The result will list only the department IDs that have more than 10 employees.

#### Common Mistakes
Incorrect Column for Grouping: Using a column that is not unique for grouping can lead to incorrect results.
Lack of HAVING Clause: Without the HAVING clause, the query would simply return all departments with their respective employee counts without filtering.
Misunderstanding GROUP BY and COUNT(*): These are crucial for correctly aggregating data.

#### Example
Use Case

Let's consider a sample table structure:

SQL
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(50),
    department_id INT
);

INSERT INTO employees (employee_id, name, department_id) VALUES
(1, 'Alice', 1), (2, 'Bob', 1), (3, 'Charlie', 1), (4, 'David', 1),
(5, 'Eve', 2), (6, 'Frank', 2), (7, 'Grace', 2), (8, 'Heidi', 2),
(9, 'Ivan', 3), (10, 'Jack', 3), (11, 'Katie', 4);

Running the query on this sample data:

SQL
SELECT department_id, COUNT(*) AS num_employees
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 10;

Output:

CODE
department_id | num_employees
--------------|---------------
1             | 4
2             | 4
3             | 3

This output is incorrect because our sample data does not contain departments with more than 10 employees. The query would return no results if the department IDs do not meet the condition.

#### Conclusion

By using GROUP BY and HAVING, we can filter and aggregate data effectively to answer questions like this. Always ensure that your grouping column is relevant and that you use the HAVING clause appropriately for filtering aggregated groups.

**Code:**

```sql
SELECT department_id, COUNT(*) AS num_employees
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 10;
```

**Example input / output:**

```text
SELECT department_id, COUNT(*) AS num_employees
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 10;

-- Output:
-- No results as there are no departments with more than 10 employees in the sample data.
```

**Data:**

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(50),
    department_id INT
);

INSERT INTO employees (employee_id, name, department_id) VALUES
(1, 'Alice', 1), (2, 'Bob', 1), (3, 'Charlie', 1), (4, 'David', 1),
(5, 'Eve', 2), (6, 'Frank', 2), (7, 'Grace', 2), (8, 'Heidi', 2),
(9, 'Ivan', 3), (10, 'Jack', 3), (11, 'Katie', 4);
```

