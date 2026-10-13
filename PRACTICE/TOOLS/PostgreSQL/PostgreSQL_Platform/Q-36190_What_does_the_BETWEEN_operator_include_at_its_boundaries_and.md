---
type: "practice"
difficulty: "Intermediate"
---

# What does the BETWEEN operator include at its boundaries and how would you filter salaries between 30000 and 50000?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ FIRST_VALUE and LAST_VALUE](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/FIRST_VALUE_and_LAST_VALUE.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To answer this question, let's first understand what the BETWEEN operator does in PostgreSQL and then see how to use it effectively.

The BETWEEN operator is used to select values within a specific range, including both boundaries. In SQL, when you write something like salary BETWEEN 30000 AND 50000, it means that the query will return all rows where the salary column value is greater than or equal to 30000 and less than or equal to 50000.

Now, let's break down how to implement this in a SQL query.

**Code:**

```sql
SELECT * FROM employees
WHERE salary BETWEEN 30000 AND 50000;

This query will return all rows where the salary column is within the range of 30000 to 50000, inclusive.
```

**Example input / output:**

```

Sample input:

SQL
Copy
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    salary INTEGER
);

INSERT INTO employees (name, salary) VALUES ('Alice', 28000), ('Bob', 35000), ('Charlie', 40000), ('David', 52000);

Sample output:

SQL
Copy
id | name   | salary
---+--------+--------
1  | Alice  | 28000
2  | Bob    | 35000
3  | Charlie| 40000
```

