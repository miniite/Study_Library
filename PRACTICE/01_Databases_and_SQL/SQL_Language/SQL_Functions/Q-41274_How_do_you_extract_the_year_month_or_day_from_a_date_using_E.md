---
type: "practice"
difficulty: "Intermediate"
---

# How do you extract the year, month, or day from a date using EXTRACT or DATEPART?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Date Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/Date_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To extract specific components (year, month, or day) from a date in SQL, we often use functions like EXTRACT or DATEPART. These functions are essential tools when working with dates and require understanding of how to query different parts of the date. Let's break down these concepts step by step.

**Code:**

```sql
-- Example using EXTRACT
SELECT EXTRACT(YEAR FROM date_column) AS year,
       EXTRACT(MONTH FROM date_column) AS month,
       EXTRACT(DAY FROM date_column) AS day
FROM your_table;

-- Example using DATEPART (common in SQL Server)
SELECT DATEPART(YEAR, date_column) AS year,
       DATEPART(MONTH, date_column) AS month,
       DATEPART(DAY, date_column) AS day
FROM your_table;
```

**Example input / output:**

```text
-- Input
SELECT EXTRACT(YEAR FROM '2023-10-05') AS year,
       DATEPART(MONTH, '2023-10-05') AS month,
       EXTRACT(DAY FROM '2023-10-05') AS day;

-- Output
+------+--------+------+
| year | month  | day  |
+------+--------+------+
| 2023 |      10|     5|
+------+--------+------+
```

**Flow:**

1. No flowchart is necessary here, but if you wanted to visualize the process, you could use:
2. Input Date
3. EXTRACT or DATEPART
4. Year
5. Month
6. Day

