---
type: "practice"
difficulty: "Basic"
---

# What are the common aggregate functions in Oracle such as COUNT, SUM, AVG, MIN, and MAX?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Oracle Aggregate Functions](<../../../../TOOLS/Oracle/Oracle_Platform/Oracle_Aggregate_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Oracle databases, understanding how to use aggregate functions is crucial for performing data analysis and generating meaningful insights. Commonly used aggregate functions in Oracle include COUNT, SUM, AVG, MIN, and MAX. These functions help summarize the dataset by providing statistical information.

**Comparison:**

| Aggregate Functions | Description                |
|---------------------|----------------------------|
| COUNT(*)            | Counts the number of rows   |
| SUM(column)         | Adds up all values in a column |
| AVG(column)         | Calculates the average value of a column |
| MIN(column)         | Finds the minimum value in a column  |
| MAX(column)         | Finds the maximum value in a column |

**Code:**

```sql
-- Example of using COUNT, SUM, AVG, MIN, and MAX functions

-- Assume we have a table named 'employees' with columns id, name, salary, department_id.
```

**Example input / output:**

```text
-- Example of using aggregate functions

SELECT COUNT(*) AS total_employees,
       SUM(salary) AS total_salary,
       AVG(salary) AS average_salary,
       MIN(salary) AS min_salary,
       MAX(salary) AS max_salary
FROM employees;

-- Output:
-- TOTAL_EMPLOYEES    TOTAL_SALARY   AVERAGE_SALARY  MIN_SALARY  MAX_SALARY
-- 3                  155000        51666.67        45000       60000
```

**Data:**

```sql
CREATE TABLE employees (
    id NUMBER,
    name VARCHAR2(100),
    salary NUMBER,
    department_id NUMBER
);
INSERT INTO employees (id, name, salary, department_id) VALUES (1, 'Alice', 50000, 10);
INSERT INTO employees (id, name, salary, department_id) VALUES (2, 'Bob', 60000, 10);
INSERT INTO employees (id, name, salary, department_id) VALUES (3, 'Charlie', 45000, 11);
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Yes
8. No
9. Yes
10. No
11. COUNT(*)
12. Is it necessary to count allrows?
13. Use COUNT(*)
14. Filter data first
15. Execute query with COUNT(*)
16. Apply filters, then useCOUNT(*) if needed
17. SUM(column)
18. Do you need the sum of aspecific column?
19. Use SUM(column)
20. Use other functions or skip thisstep
21. AVG(column)
22. Is averaging important for youranalysis?
23. Calculate average using AVG()
24. Use other functions or skip thisstep
25. MIN(column)
26. Do you need to find theminimum value in a column?
27. Use MIN(column)
28. Use other functions or skip thisstep
29. MAX(column)
30. Do you need the maximumvalue in a column?
31. Use MAX(column)
32. Use other functions or skip thisstep

