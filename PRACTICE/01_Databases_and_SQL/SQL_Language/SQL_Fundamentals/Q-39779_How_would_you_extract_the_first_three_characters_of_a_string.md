---
type: "practice"
difficulty: "Basic"
---

# How would you extract the first three characters of a string column using SUBSTRING?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ String Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/String_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To extract the first three characters from a string column in SQL, we use the SUBSTRING function. This function is straightforward and allows us to specify the starting position and the number of characters to return from a given string. The syntax typically looks like this:

SQL
SELECT SUBSTRING(column_name FROM start_position FOR length)
FROM table_name;

The interviewer is likely probing your understanding of basic SQL functions and how to manipulate strings within queries. They may also be checking if you can correctly apply the function or make common mistakes, such as starting from an incorrect position.

Let's break down the solution step by step:

Identify the column: Determine which column in your table contains the string.
Determine the start position and length: For extracting the first three characters, the start_position is 1 (the default for SQL when no starting position is specified), and the length is 3.

Here’s a worked example:

Suppose we have a table called employees, with a column named name. We want to extract the first three characters of each name.

SQL
SELECT SUBSTRING(name FROM 1 FOR 3) AS first_three_chars
FROM employees;

This query will return a new column first_three_chars containing the first three characters from each value in the name column.

#### Common Mistakes
and Follow-Ups

Starting Position: A common mistake is misunderstanding that SQL typically starts counting positions at 1, not 0. For example:

SQL
-- Incorrect: returns the first four characters
SELECT SUBSTRING(name FROM 2 FOR 4) AS first_three_chars;

Length Issue: Another mistake might be setting an incorrect length value. For instance:

SQL
-- Incorrect: returns more than three characters
SELECT SUBSTRING(name FROM 1 FOR 5) AS first_three_chars;

By understanding these concepts and avoiding common pitfalls, you can effectively use the SUBSTRING function in SQL.

**Comparison:**

| id | name   | first_three_chars |
|----|--------|------------------|
| 1  | Alice  | Alik             |
| 2  | Bob    | Bobs             |
| 3  | Charlie| Charl            |

**Code:**

```sql
-- Example query to extract first three characters of a column named 'name' from table 'employees'
SELECT SUBSTRING(name FROM 1 FOR 3) AS first_three_chars
FROM employees;
```

**Example input / output:**

| id | name   | first_three_chars |
|----|--------|------------------|
| 1  | Alice  | Alik             |
| 2  | Bob    | Bobs             |
| 3  | Charlie| Charl            |

**Data:**

```json
{
  "employees": [
    {"id": 1, "name": "Alice"},
    {"id": 2, "name": "Bob"},
    {"id": 3, "name": "Charlie"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Define the column and startposition
4. Is the start_position 1?
5. Use SUBSTRING with length 3
6. Adjust start_position to 1
7. Execute query
8. Return result

