---
type: "practice"
difficulty: "Basic"
---

# What is the difference between LTRIM and RTRIM?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ String Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/String_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Let's break down the key differences between LTRIM and RTRIM in SQL functions, which are primarily used to remove leading or trailing spaces from strings.

#### Triaging the Question

The core of this question is understanding how these two string manipulation functions work. LTRIM stands for "Left Trim," while RTRIM stands for "Right Trim." The interviewer is probing your basic knowledge of SQL string functions and your ability to differentiate between similar concepts.

#### First-Principles Solution

LTRIM Function:

Purpose: LTRIM removes leading spaces (spaces at the beginning) from a given string.
Syntax Example:
SQL
SELECT LTRIM('  Hello, World! ') AS TrimmedString;
Expected Output: 'Hello, World! '

RTRIM Function:

Purpose: RTRIM removes trailing spaces (spaces at the end) from a given string.
Syntax Example:
SQL
SELECT RTRIM('Hello, World!  ') AS TrimmedString;
Expected Output: 'Hello, World!'

Combining LTRIM and RTRIM:

If you need to remove both leading and trailing spaces, you can use a combination of LTRIM and RTRIM. For instance:
SQL
SELECT RTRIM(LTRIM('  Hello, World!  ')) AS TrimmedString;
Expected Output: 'Hello, World!'

#### Closing Summary
+ Common Mistakes/Follow-Ups:
Both LTRIM and RTRIM are essential for cleaning up strings to ensure accurate data processing.
A common mistake is not understanding the difference between leading and trailing spaces. Always remember that LTRIM targets the beginning of a string, while RTRIM targets the end.
During follow-ups, you might be asked about scenarios where both functions are used together or how they interact with other string functions like TRIM.

**Code:**

```sql
-- Example usage
SELECT
    LTRIM('  Hello, World! ') AS LeftTrimmed,
    RTRIM('Hello, World!  ') AS RightTrimmed,
    TRIM('  Hello, World!  ') AS BothTrimmed; -- Using the TRIM function for both leading and trailing spaces
```

**Example input / output:**

```text
+--------------+-------------+------------+
| LeftTrimmed  | RightTrimmed| BothTrimmed|
+--------------+-------------+------------+
| Hello, World!| Hello, World!|Hello, World!|
+--------------+-------------+------------+
```

