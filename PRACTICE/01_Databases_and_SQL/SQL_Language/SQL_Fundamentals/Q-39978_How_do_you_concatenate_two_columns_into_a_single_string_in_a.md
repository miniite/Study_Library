---
type: "practice"
difficulty: "Basic"
---

# How do you concatenate two columns into a single string in a query, and how do NULLs affect it?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ String Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/String_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When concatenating two columns into a single string in SQL, the main goal is to join their values with some delimiter (like a space or comma). This operation can be performed using specific functions depending on your database system. For instance, in PostgreSQL and MySQL, you would use CONCAT or ||, while in SQL Server, you might use +. However, handling NULLs is crucial to avoid unexpected results.

**Code:**

```sql
-- Example in PostgreSQL
SELECT CONCAT(column1, ' ', column2) AS concatenated_string FROM your_table;

-- Example in MySQL
SELECT CONCAT(column1, ' ', column2) AS concatenated_string FROM your_table;

-- Example in SQL Server
SELECT column1 + ' ' + ISNULL(column2, '') AS concatenated_string FROM your_table;
```

**Example input / output:**

```sql
-- Input (example data)
CREATE TABLE example_table (
    id INT,
    first_name VARCHAR(50),
    last_name VARCHAR(50)
);

INSERT INTO example_table (id, first_name, last_name) VALUES
(1, 'John', 'Doe'),
(2, NULL, 'Smith'),
(3, 'Jane', NULL),
(4, NULL, NULL);

-- Output
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM example_table;

Output:

full_name
John Doe
(NULL)
Jane
(NULL)

Note the use of ISNULL in SQL Server to handle NULLs.
```

