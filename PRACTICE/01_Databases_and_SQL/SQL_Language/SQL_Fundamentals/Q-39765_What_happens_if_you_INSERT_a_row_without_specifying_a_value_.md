---
type: "practice"
difficulty: "Basic"
---

# What happens if you INSERT a row without specifying a value for a NOT NULL column that has no default?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ NULL Handling](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/NULL_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When inserting a row into a table in SQL, if you attempt to insert a value into a NOT NULL column but do not specify a value for it, the query will fail due to a constraint violation. This is because the NOT NULL constraint ensures that every row must have a non-null (non-empty) value for that column.

#### Triaging the Question

The core of this question revolves around understanding how SQL handles the NOT NULL constraint when you try to insert data without providing a value for it. The key idea here is that SQL requires all columns with a NOT NULL constraint to have a non-null value, and failing to provide one will result in an error.

#### First-Principles Solution

Let's break this down from the ground up:

Understanding NOT NULL Constraint: A column with the NOT NULL constraint cannot store any null values.
Insert Statement Without Value: If you attempt to insert a row where you do not provide a value for a NOT NULL column, SQL will throw an error because it violates the integrity of the table.

#### Example

#### Cons
ider a simple table called Employees with columns id, name, and age. The id column is set as NOT NULL.

SQL
CREATE TABLE Employees (
    id INT NOT NULL,
    name VARCHAR(100),
    age INT
);

If you try to insert an employee without specifying the id, the following query will fail:

SQL
INSERT INTO Employees (name, age) VALUES ('Alice', 30);

This query will raise a constraint violation error because the NOT NULL column id does not have a specified value.

#### Common Mistakes
and Follow-Ups
Forgetting to Specify Values: A common mistake is forgetting to provide values for all columns, especially those with NOT NULL constraints.
Ignoring Default Values: Another pitfall is assuming that default values will automatically be used if no value is provided. For columns without a default value, specifying the column name and its value is mandatory.

#### Closing Summary

In summary, when you attempt to insert a row into an SQL table with a NOT NULL column and do not provide a value for it, the operation will fail due to a constraint violation. Always ensure that all required columns are properly populated to avoid such errors.

