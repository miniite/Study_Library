---
type: "practice"
difficulty: "Deep"
---

# How would you find the median value of a column in MySQL?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Ordering and Limits](<../../../../TOOLS/MySQL/MySQL_Platform/Ordering_and_Limits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To find the median value of a column in MySQL, we need to understand that the median is the middle value when all values are sorted. For an odd number of entries, it's simply the middle one; for an even number, it's the average of the two middle ones. Here’s how we can achieve this:

#### Triaging the Question
The core of the question is to implement a query that correctly handles both even and odd numbers of rows in a column. We also need to consider performance implications since sorting large datasets can be costly.
First Principles Approach:
Sort the Data: First, we sort the data based on the column in question.
Handle Odd and Even Cases: Depending on whether the number of entries is odd or even, we select the middle value(s) appropriately.

Let’s break down these steps with a concrete example:

Step-by-Step Solution:

Prepare Sample Data:
SQL
CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    salary DECIMAL(10,2)
);

INSERT INTO employees (salary) VALUES (5000), (6000), (7000), (8000), (9000);
Query for Median Salary:
SQL
-- Step 1: Calculate the total number of rows and the offset to find the middle row(s)
WITH count_rows AS (
    SELECT COUNT(*) as total_count FROM employees
),
median_offset AS (
    SELECT FLOOR(total_count / 2) + 1 AS median_offset FROM count_rows
)
-- Step 2: Select the median value(s) based on the offset
SELECT AVG(salary) as median_salary
FROM (
    SELECT salary, ROW_NUMBER() OVER (ORDER BY salary ASC) as row_num
    FROM employees
) subquery
WHERE row_num = median_offset OR row_num = median_offset + 1;

#### Closing Summary
and Common Mistakes:

The query above handles both odd and even numbers of rows correctly by using a window function (ROW_NUMBER()) to assign a rank to each salary.
For an odd number of entries, we select the single middle row’s value directly.
For an even number, we average the two middle rows’ values.

A common mistake is not handling the odd and even cases correctly. Always ensure you have the correct logic to handle both scenarios to get accurate results.

**Comparison:**

| id | salary |
|----|--------|
| 1  | 5000   |
| 2  | 6000   |
| 3  | 7000   |
| 4  | 8000   |
| 5  | 9000   |

**Code:**

```sql
-- Sample data creation and insertion
CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    salary DECIMAL(10,2)
);

INSERT INTO employees (salary) VALUES (5000), (6000), (7000), (8000), (9000);
```

**Example input / output:**

```text
-- Input Query for Median Salary Calculation
WITH count_rows AS (
    SELECT COUNT(*) as total_count FROM employees
),
median_offset AS (
    SELECT FLOOR(total_count / 2) + 1 AS median_offset FROM count_rows
)
SELECT AVG(salary) as median_salary
FROM (
    SELECT salary, ROW_NUMBER() OVER (ORDER BY salary ASC) as row_num
    FROM employees
) subquery
WHERE row_num = median_offset OR row_num = median_offset + 1;
```

**Data:**

```sql
-- Sample data creation and insertion
CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    salary DECIMAL(10,2)
);

INSERT INTO employees (salary) VALUES (5000), (6000), (7000), (8000), (9000);
```

**Flow:**

1. Yes
2. No
3. Input
4. Count rows
5. Select median offset
6. Generate subquery with rownumbers
7. Is count odd?
8. Avg salary from middle tworows
9. SINGLE MIDDLE ROW'S VALUE

