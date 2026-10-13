---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between CONCAT and CONCAT_WS?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ String Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/String_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with SQL, CONCAT and CONCAT_WS are both used to join strings together. However, they have some key differences that make them suitable for different use cases.

#### Triaging the Question

The interviewer is probing your understanding of string manipulation functions in SQL, specifically focusing on how CONCAT and CONCAT_WS differ. The core idea here is to understand when each function would be more appropriate based on their behavior with NULL values and performance implications.

#### First-Principles Solution
1. Understanding CONCAT
Syntax: CONCAT(str1, str2, ..., str_n)
Behavior: CONCAT simply concatenates the strings provided as arguments.
NULL Handling: If any of the arguments are NULL, the result is NULL.
2. Understanding CONCAT_WS
Syntax: CONCAT_WS(wsep, str1, str2, ..., str_n)
Behavior: CONCAT_WS concatenates the strings provided as arguments and inserts a separator (wsep) between each pair of strings.
NULL Handling: If any of the arguments (except the first one) are NULL, those arguments are skipped. The result is only affected if all non-argument values after the first argument are NULL.

#### Example
Walkthrough

Let's consider an example where we want to combine multiple columns in a table:

SQL
-- Using CONCAT
SELECT CONCAT(column1, ' ', column2, ' ', column3) FROM my_table;

-- Using CONCAT_WS
SELECT CONCAT_WS(' - ', NULL, 'John', NULL, 'Doe') AS combined_name;

In the CONCAT example, if any of the columns are NULL, they will be included in the result. However, in the CONCAT_WS example, the second and fourth values ('John' and 'Doe') are concatenated with a separator ' - ', while the first and third values (both NULL) are ignored.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, use CONCAT when you need to concatenate strings without worrying about skipping any parts of the string. Use CONCAT_WS when you want to specify a separator between each pair of non-NULL arguments and ensure that NULL values are ignored.

#### Common mistakes
include:

Misunderstanding NULL Handling: Forgetting that CONCAT_WS skips NULLs, while CONCAT includes them.
Choosing the Wrong Function: Using CONCAT when a separator is required, or vice versa.
Depth + Language Match

Given your intermediate experience as a Database Developer 1/2, it's important to understand both functions' behavior and their practical applications in real-world scenarios.

**Code:**

```sql
-- Example of CONCAT usage
SELECT CONCAT(column1, ' ', column2, ' ', column3) FROM my_table;

-- Example of CONCAT_WS usage
SELECT CONCAT_WS(' - ', NULL, 'John', NULL, 'Doe') AS combined_name;
```

**Example input / output:**

```

-- Input for CONCAT

column1	column2	column3
NULL	'John'	'Doe'

-- Output for CONCAT ' NULL John Doe '

-- Input for CONCAT_WS NULL, 'John', NULL, 'Doe'

CODE
Copy
```sql
-- Output for CONCAT_WS with separator '- '
' - John - Doe '
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. CONCAT
6. Is there a NULL?
7. Return NULL
8. Concatenate all strings
9. CONCAT_WS
10. Is there a NULL (except first)?
11. Skip this value
12. Insert separator andconcatenate remaining values

