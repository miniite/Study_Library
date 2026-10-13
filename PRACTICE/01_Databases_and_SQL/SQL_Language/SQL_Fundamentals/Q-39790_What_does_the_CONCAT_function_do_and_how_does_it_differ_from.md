---
type: "practice"
difficulty: "Basic"
---

# What does the CONCAT function do and how does it differ from using the || or + concatenation operator?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ String Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/String_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The CONCAT function in SQL is used to concatenate, or join together, multiple strings into a single string. It differs from the || and + operators primarily in its handling of NULL values and its syntax.

To start with the basics, the || operator and the + operator are both commonly used for string concatenation across SQL dialects. However, their behavior can vary slightly depending on the database system you're working with. For instance, in PostgreSQL, both operators behave similarly when dealing with NULL values; they return NULL if any of the operands is NULL.

On the other hand, the CONCAT function provides a more standardized approach to string concatenation across different SQL dialects and is designed to handle NULL values consistently. It always returns an empty string if any of its arguments are NULL, which can be beneficial for avoiding errors that might occur with the || or + operators.

Here's a step-by-step explanation:

Syntax: The basic syntax for using CONCAT is:

SQL
CONCAT(str1, str2, ...)

For example:

SQL
SELECT CONCAT('Hello', ' ', 'World');

NULL Handling:

With the || or + operator: If any of the operands is NULL in PostgreSQL, the result will also be NULL.
SQL
SELECT 'Hello' || NULL;  -- Result: NULL
Using CONCAT: The function returns an empty string if any argument is NULL.
SQL
SELECT CONCAT('Hello', NULL);  -- Result: ''

#### Example
Let's consider a scenario where you have the following data:

SQL
CREATE TABLE greetings (
    name VARCHAR(100),
    greeting VARCHAR(100)
);

INSERT INTO greetings (name, greeting) VALUES ('Alice', 'Good morning'), ('Bob', NULL), ('Charlie', 'Good evening');

Using ||:

SQL
SELECT name || ' ' || greeting AS full_greeting FROM greetings;

This might yield unexpected results due to the NULL value in the greeting column.

Using CONCAT:

SQL
SELECT CONCAT(name, ' ', greeting) AS full_greeting FROM greetings;

This will return a cleaner result by handling the NULLs appropriately.

In summary, while both || and + can be used for string concatenation in SQL, using CONCAT ensures more predictable behavior when dealing with NULL values. It's generally recommended to use CONCAT for better portability and consistent results across different databases.

