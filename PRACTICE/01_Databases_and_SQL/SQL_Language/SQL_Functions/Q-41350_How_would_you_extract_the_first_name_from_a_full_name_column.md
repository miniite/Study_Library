---
type: "practice"
difficulty: "Deep"
---

# How would you extract the first name from a full name column that has a space separator?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ String Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/String_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To extract the first name from a full_name column where names are separated by spaces, we need to focus on using SQL string functions effectively. This task involves identifying and isolating the portion of the string before the first space. Let's break down the solution step-by-step.

#### Triage

The question is asking how to split a full name into its constituent parts based on a common separator (a single space). The interviewer is likely probing your understanding of SQL string manipulation functions and your ability to write concise, efficient queries.

#### First-Principles Solution

#### Assumptions

The full_name column contains names separated by a single space.
The first name does not contain any spaces (if it did, additional logic would be required).

SQL String Functions:

We can use the SUBSTRING_INDEX function in MySQL or similar functions in other SQL dialects to extract the first part of the string before the first space.

#### Example
Solution

In MySQL, you can use SUBSTRING_INDEX as follows:

SQL
SELECT SUBSTRING_INDEX(full_name, ' ', 1) AS first_name
FROM employees;

This query works by:

SUBSTRING_INDEX(full_name, ' ', 1) splits the full_name at the first space and returns everything before it.
The result is aliased as first_name.

#### Closing Summary
+ Common Mistakes/Follow-Ups

The solution leverages a straightforward SQL function to achieve the goal. However, there are edge cases to consider:

If names can contain multiple spaces (e.g., "John Doe"), additional logic might be needed.
In other SQL dialects, you might need to use different functions like LEFT, TRIM, or custom string manipulation functions.

#### Common Mistakes
Misunderstanding the function: Using incorrect functions or incorrectly specifying arguments.
Edge cases not considered: Not handling names with multiple spaces correctly.
Ignoring case sensitivity: Ensuring that the function handles all possible input formats.

If asked further, you could discuss performance considerations, potential issues with very long names, and how to handle special characters in names.

**Code:**

```sql
SELECT SUBSTRING_INDEX(full_name, ' ', 1) AS first_name
FROM employees;
```

**Example input / output:**

```text
-- Input
SELECT SUBSTRING_INDEX(full_name, ' ', 1) AS first_name
FROM employees;

-- Output
+--------------+
| first_name   |
+--------------+
| John         |
| Jane         |
| Michael      |
+--------------+
```

**Data:**

```json
{
  "employees": [
    {"full_name": "John Doe"},
    {"full_name": "Jane Smith"},
    {"full_name": "Michael Johnson"}
  ]
}
```

