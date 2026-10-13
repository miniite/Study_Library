---
type: "practice"
difficulty: "Basic"
---

# What is the difference between a WHERE clause using AND versus OR when combining conditions?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Join Types](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Join_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with SQL, understanding how AND and OR clauses interact in your WHERE condition is crucial. The core idea here is that these operators determine which rows from a table satisfy multiple conditions.

In essence, the difference between using AND versus OR lies in how they combine multiple predicates to filter the data:

AND: It returns records where all the conditions are true.
OR: It returns records where at least one of the conditions is true.

Let's dive into a step-by-step explanation from first principles, then cover some common mistakes and follow-ups.

**Comparison:**

```
AND Output	OR Output
Jane Smith	Michael Brown
	Jane Smith (again)
```

**Code:**

```sql
-- Example using AND in WHERE clause
SELECT * FROM employees
WHERE department = 'Sales' AND salary > 50000;

-- Example using OR in WHERE clause
SELECT * FROM employees
WHERE department = 'Sales' OR salary > 50000;
```

**Example input / output:**

```

Given the sample data:

Using AND:

SQL
Copy
SELECT * FROM employees
WHERE department = 'Sales' AND salary > 50000;

Expected output (only one row):

JSON
Copy
[
  {"id": 2, "name": "Jane Smith", "department": "Sales", "salary": 55000}
]

Using OR:

SQL
Copy
SELECT * FROM employees
WHERE department = 'Sales' OR salary > 50000;

Expected output (multiple rows):

JSON
Copy
[
  {"id": 2, "name": "Jane Smith", "department": "Sales", "salary": 55000},
  {"id": 4, "name": "Michael Brown", "department": "Sales", "salary": 65000}
]
```

**Data:**

```json
{
  "employees": [
    {"id": 1, "name": "John Doe", "department": "Engineering", "salary": 60000},
    {"id": 2, "name": "Jane Smith", "department": "Sales", "salary": 55000},
    {"id": 3, "name": "Emily Johnson", "department": "HR", "salary": 45000},
    {"id": 4, "name": "Michael Brown", "department": "Sales", "salary": 65000}
  ]
}
```

**Flow:**

1. Condition 1 AND Condition 2
2. left
3. Condition 1 OR Condition 2
4. left
5. Include row
6. Skip row
7. Include row
8. Skip row

