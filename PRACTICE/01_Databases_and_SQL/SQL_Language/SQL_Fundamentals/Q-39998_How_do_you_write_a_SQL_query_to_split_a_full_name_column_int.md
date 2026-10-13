---
type: "practice"
difficulty: "Basic"
---

# How do you write a SQL query to split a full_name column into first_name and last_name during transformation?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ String Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/String_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To split a full_name column into first_name and last_name, we need to use SQL string functions. The basic idea is to utilize functions that can help us identify and separate parts of a full name, such as the position or delimiters used in names.

#### Triaging the question

Question: How do you write an SQL query to split a full_name column into first_name and last_name?
What is being probed: Understanding of string manipulation functions in SQL (e.g., SUBSTRING, LEN, CHARINDEX) and their usage.

#### Assumptions
We assume that the full_name column contains names separated by a space. If this isn't the case, additional logic or pattern matching may be required.

Step-by-step solution:

Identify the position of the delimiter (space): Use CHARINDEX to find the position of the first space in the full_name.
Extract the first name: Use SUBSTRING with a starting index from 0 to the position identified by CHARINDEX.
Extract the last name: Use another SUBSTRING after the first space (position + 1) to the end of the string.

#### First-principles solution

SQL
-- Sample SQL query to split full_name into first_name and last_name
SELECT
    full_name,
    SUBSTRING(full_name, 0, CHARINDEX(' ', full_name)) AS first_name,
    CASE
        WHEN LEN(full_name) > CHARINDEX(' ', full_name) + 1 THEN
            SUBSTRING(full_name, CHARINDEX(' ', full_name) + 1, LEN(full_name))
        ELSE 'N/A'
    END AS last_name
FROM your_table;

#### Closing summary
and common mistakes/follow-ups:

Summary: The query uses CHARINDEX to find the position of the first space in the full_name, then splits it into first_name and last_name using SUBSTRING. It handles cases where the name might not contain a middle name or initial.

#### Common Mistakes
Not considering edge cases like names without spaces (e.g., "JohnDoe").
Forgetting to handle cases where multiple delimiters exist, which would require more complex logic.

This approach ensures that the full_name is correctly split into meaningful parts. However, for a more robust solution, additional checks and handling of various name formats might be necessary.

**Code:**

```sql
-- Example SQL query to split full_name into first_name and last_name
SELECT
    full_name,
    SUBSTRING(full_name, 0, CHARINDEX(' ', full_name)) AS first_name,
    CASE
        WHEN LEN(full_name) > CHARINDEX(' ', full_name) + 1 THEN
            SUBSTRING(full_name, CHARINDEX(' ', full_name) + 1, LEN(full_name))
        ELSE 'N/A'
    END AS last_name
FROM your_table;
```

