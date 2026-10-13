---
type: "practice"
difficulty: "Basic"
---

# Do aggregate functions ignore NULL values? Explain which ones do and which do not.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Aggregate Precision and Types](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/Aggregate_Precision_and_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview
Aggregate functions in SQL are used to perform calculations on a set of values and return a single value as the result. One common misconception is that all aggregate functions ignore NULL values, but this isn't always true. In this answer, we'll explore which aggregate functions do and don't ignore NULL values and why.

#### Triage
The core of the question revolves around understanding the behavior of SQL aggregate functions in the presence of NULL values. The key idea here is to identify the specific functions that handle NULLs differently from others.

Aggregate functions can be categorized into two types based on their handling of NULL values:

Aggregates that ignore NULLs: These include COUNT, SUM, AVG, MAX, and MIN.
Aggregates that do not ignore NULLs: This includes GROUP_CONCAT in certain SQL flavors like MySQL.

Step-by-Step Solution:

Aggregate Functions Ignoring NULL Values

COUNT: Returns the number of items. It counts only non-NULL values. For example, if you have a table with 5 rows, but one has a NULL value for a column, COUNT will still count this row.

SQL
SELECT COUNT(column_name) FROM table_name;

SUM: Adds up numeric columns and ignores any NULLs in the process.

SQL
SELECT SUM(column_name) FROM table_name;

AVG (AVERAGE): Computes the average of all non-NULL values in a column.

SQL
SELECT AVG(column_name) FROM table_name;

MAX: Returns the maximum value from the selected columns, ignoring NULLs.

SQL
SELECT MAX(column_name) FROM table_name;

MIN: Similarly, returns the minimum value while ignoring NULLs.

SQL
SELECT MIN(column_name) FROM table_name;
Aggregate Functions Not Ignoring NULL Values
GROUP_CONCAT: This function in MySQL combines values from a group into a single concatenated string and treats NULL as an empty string rather than ignoring it.
SQL
SELECT GROUP_CONCAT(column_name SEPARATOR ', ') AS concatenated_values FROM table_name;
Why These Differences Matter

Understanding these differences is crucial because different SQL databases (like MySQL, PostgreSQL, SQL Server) may handle NULLs in their own ways. For example, the behavior of GROUP_CONCAT might differ between MySQL and other database systems.

#### Common Mistakes
and Follow-Ups

Mistake 1: Assuming all aggregate functions ignore NULL values.

#### Follow-Up
Ask about specific functions to confirm understanding, e.g., "What happens when using GROUP_CONCAT in a query?"

Mistake 2: Not considering the implications of ignoring or not ignoring NULLs in complex queries.

#### Follow-Up
Discuss how this affects performance and data accuracy.

#### Closing Summary

In summary, it's important to know which aggregate functions ignore NULL values (COUNT, SUM, AVG, MAX, MIN) versus those that do not (GROUP_CONCAT). Understanding these nuances is critical for writing robust SQL queries and avoiding unexpected results.

**Comparison:**

| Column Name | Description           |
|-------------|-----------------------|
| COUNT       | Ignores NULLs         |
| SUM         | Ignores NULLs         |
| AVG         | Ignores NULLs         |
| MAX         | Ignores NULLs         |
| MIN         | Ignores NULLs         |
| GROUP_CONCAT| Treats NULL as empty  |

**Code:**

```sql
-- Example of COUNT ignoring NULLs
SELECT COUNT(column_name) FROM table_name;

-- Example of GROUP_CONCAT treating NULL as empty string
SELECT GROUP_CONCAT(column_name SEPARATOR ', ') AS concatenated_values FROM table_name;
```

**Example input / output:**

```text
-- Input and Expected Output for COUNT
SELECT COUNT(column_name) FROM table_name;
-- Expected: If there are 5 rows, it returns 4 if one row has a NULL.

-- Input and Expected Output for GROUP_CONCAT
SELECT GROUP_CONCAT(column_name SEPARATOR ', ') AS concatenated_values FROM table_name;
-- Expected: "1,3"
```

**Data:**

```json
[
  {
    "column_name": 1,
    "column_name2": null
  },
  {
    "column_name": 3,
    "column_name2": 4
  }
]
```

**Flow:**

1. Yes
2. No
3. Yes
4. Yes
5. MySQL
6. Other DBMS
7. Query with COUNT
8. Aggregate Function?
9. Ignore NULLs?
10. NULL Ignored
11. NULL Included
12. Query with GROUP_CONCAT
13. Aggregate Function?
14. Group_Concat Behavior?
15. Treat NULL as empty string
16. Handle NULLs Differently

