---
type: "practice"
difficulty: "Deep"
---

# What is the difference between RANK and DENSE_RANK window functions?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Window Functions Overview](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/Window_Functions_Overview.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In SQL, RANK and DENSE_RANK are both window functions used to assign a unique rank to rows within a result set that has been divided into partitions according to some criteria. However, they differ in how ties (rows with the same value) are handled.

#### Triage
Understanding RANK vs DENSE_RANK
What the Question is Asking: The core of this question lies in understanding the nuances between RANK and DENSE_RANK functions.
Key Assumptions: Both functions are used for ranking data, but they behave differently when there are duplicate values in the dataset.
Reasoning from First Principles:
Both functions use a window partition to rank rows within each partition based on an order by clause.
The main difference arises when there are ties (rows with the same value) in the ranking criteria.
Detailed Explanation
RANK Function

RANK() assigns a unique rank to every row, even if they have the same value. If two or more rows have the same value according to the order by clause, each of them is given the next highest rank that would have been assigned had there been no ties. For example, in a list of employee salaries, if two employees have the same salary, RANK() will give both the 2nd rank instead of 1st and 2nd.

DENSE_RANK Function

DENSE_RANK() also assigns unique ranks but does not skip numbers when there are ties. If two or more rows have the same value according to the order by clause, they receive the same rank, and the next highest rank is assigned immediately after these tied values. For example, in a list of employee salaries, if two employees have the same salary, both will get the 1st rank, and the next employee with a different salary will be given the 2nd rank.

#### Example

Let's consider an employees table with columns id, name, and salary.

id	name	salary
1	Alice	50000
2	Bob	45000
3	Charlie	50000
4	David	60000
RANK Function Query
SQL
SELECT id, name, salary,
       RANK() OVER (ORDER BY salary DESC) as rank_salary
FROM employees;

Output:

id	name	salary	rank_salary
4	David	60000	1
1	Alice	50000	2
3	Charlie	50000	2
2	Bob	45000	3
DENSE_RANK Function Query
SQL
SELECT id, name, salary,
       DENSE_RANK() OVER (ORDER BY salary DESC) as dense_rank_salary
FROM employees;

Output:

id	name	salary	dense_rank_salary
4	David	60000	1
1	Alice	50000	2
3	Charlie	50000	2
2	Bob	45000	3

#### Common Mistakes
and Follow-Ups
Misunderstanding Ties: Many developers confuse RANK with ROW_NUMBER, which assigns a unique number to each row within a partition without skipping numbers.
Ignoring the Order by Clause: The order in which rows are ranked is determined by the ORDER BY clause, so it's crucial to specify this correctly.
Misusing Window Functions: Always ensure that window functions like RANK and DENSE_RANK are used within appropriate partitions.

In summary, while both RANK and DENSE_RANK are powerful tools for ranking data in SQL, understanding their differences is key to using them effectively.

**Comparison:**

| id | name   | salary | dense_rank_salary |
| 1  | Alice  | 50000  | 2                 |
| 2  | Bob    | 45000  | 3                 |
| 3  | Charlie| 50000  | 2                 |
| 4  | David  | 60000  | 1                 |

**Code:**

```sql
-- Example RANK function query
SELECT id, name, salary,
       RANK() OVER (ORDER BY salary DESC) as rank_salary
FROM employees;

-- Example DENSE_RANK function query
SELECT id, name, salary,
       DENSE_RANK() OVER (ORDER BY salary DESC) as dense_rank_salary
FROM employees;
```

**Example input / output:**

```text
Input:
-- Example RANK function query
SELECT id, name, salary,
       RANK() OVER (ORDER BY salary DESC) as rank_salary
FROM employees;

Output:
id | name   | salary | rank_salary
4  | David  | 60000  | 1
1  | Alice  | 50000  | 2
3  | Charlie| 50000  | 2
2  | Bob    | 45000  | 3

Input:
-- Example DENSE_RANK function query
SELECT id, name, salary,
       DENSE_RANK() OVER (ORDER BY salary DESC) as dense_rank_salary
FROM employees;

Output:
id | name   | salary | dense_rank_salary
4  | David  | 60000  | 1
1  | Alice  | 50000  | 2
3  | Charlie| 50000  | 2
2  | Bob    | 45000  | 3
```

**Data:**

```plaintext
id | name   | salary
1  | Alice  | 50000
2  | Bob    | 45000
3  | Charlie| 50000
4  | David  | 60000
```

**Flow:**

1. Yes
2. No
3. Start
4. Are there ties?
5. DENSE_RANK()
6. RANK()
7. Output ranked data
8. Output ranked data with gaps

