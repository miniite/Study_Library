---
type: "practice"
difficulty: "Deep"
---

# Write a query to list departments whose average salary is above the company-wide average salary.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Subquery Practice Problems](<../../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Subquery_Practice_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to find departments where the average salary of their employees is higher than the overall average salary in the company. This requires a couple of steps:

Calculate the overall average salary.
Find departments whose average employee salary exceeds that value.

Let's break it down step by step:

#### Triage

The interviewer is probing your ability to use subqueries and joins effectively. The core idea here involves calculating an aggregate (average) value within a dataset, comparing it with another aggregated value, and filtering based on the comparison. We will need to ensure that our query handles large datasets efficiently.

#### First Principles Solution
Calculate the company-wide average salary:
This can be done by summing all salaries and dividing by the number of employees.
Find departments with an average salary greater than this value:
For each department, calculate its average salary using a subquery.
Compare this departmental average to the overall average.
Full Solution

First, we'll start by calculating the company-wide average salary:

SQL
WITH OverallAverage AS (
    SELECT AVG(salary) as avg_salary
    FROM employees
)

Next, we will use this result in a subquery to find departments with higher averages:

SQL
SELECT
    department_id,
    AVG(salary) as dept_avg_salary
FROM
    (SELECT department_id, salary, (SELECT avg_salary FROM OverallAverage) as overall_avg_salary FROM employees) AS DeptSalaries
GROUP BY
    department_id
HAVING
    AVG(salary) > (SELECT avg_salary FROM OverallAverage);

#### Closing Summary
+ Common Mistakes/Follow-Ups

The solution is straightforward but requires careful handling to ensure accuracy and efficiency. A common mistake could be not properly aliasing the subquery results, which can lead to performance issues or syntax errors.

#### Follow-ups

What if the data set is very large? How would you optimize this query?
Can you modify the query to return a list of departments with their average salaries, sorted in descending order?

**Comparison:**

| Department ID | Average Salary |
|---------------|----------------|
| 1             | 5500.00        |
| 2             | 5000.00        |
| 3             | 6500.00        |

**Code:**

```sql
WITH OverallAverage AS (
    SELECT AVG(salary) as avg_salary
    FROM employees
)
SELECT
    department_id,
    AVG(salary) as dept_avg_salary
FROM
    (SELECT department_id, salary, (SELECT avg_salary FROM OverallAverage) as overall_avg_salary FROM employees) AS DeptSalaries
GROUP BY
    department_id
HAVING
    AVG(salary) > (SELECT avg_salary FROM OverallAverage);
```

**Example input / output:**

```text
WITH OverallAverage AS (
    SELECT AVG(salary) as avg_salary
    FROM employees
)
SELECT
    department_id,
    AVG(salary) as dept_avg_salary
FROM
    (SELECT department_id, salary, (SELECT avg_salary FROM OverallAverage) as overall_avg_salary FROM employees) AS DeptSalaries
GROUP BY
    department_id
HAVING
    AVG(salary) > (SELECT avg_salary FROM OverallAverage);

-- Sample Output:
+------------+----------------+
| department_id | dept_avg_salary |
+------------+----------------+
| 3           | 6500.00        |
+------------+----------------+
```

**Data:**

```json
{
  "employees": [
    {"employee_id": 1, "name": "John", "salary": 5000, "department_id": 1},
    {"employee_id": 2, "name": "Jane", "salary": 6000, "department_id": 1},
    {"employee_id": 3, "name": "Mike", "salary": 4800, "department_id": 2},
    {"employee_id": 4, "name": "Lily", "salary": 5200, "department_id": 2},
    {"employee_id": 5, "name": "Tom", "salary": 6500, "department_id": 3}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Calculate Overall AverageSalary
4. Is Department Average >Overall Average?
5. Select Departments withHigher Average
6. No Departments Meet Criteria
7. Return Departments withHigher Avg. Salary

