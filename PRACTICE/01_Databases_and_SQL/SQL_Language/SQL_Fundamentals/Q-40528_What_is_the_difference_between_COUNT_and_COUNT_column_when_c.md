---
type: "practice"
difficulty: "Basic"
---

# What is the difference between COUNT(*) and COUNT(column) when counting valid observations for a feature?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ NULL Handling](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/NULL_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the difference between COUNT(*) and COUNT(column) in SQL, let's start with a brief overview. COUNT(*) is used to count all rows in a table, regardless of whether they are null or contain valid data. On the other hand, COUNT(column) counts only the non-null values in that specific column.

When counting valid observations for a feature, using COUNT(column) ensures you're only including entries where the feature has actual data. Conversely, COUNT(*) will count all rows, even those with null values in your specified column.

Here’s how to reason through this from first principles:

Understanding the Syntax:

SELECT COUNT(*) FROM table_name;: This returns the total number of rows in the table.
SELECT COUNT(column_name) FROM table_name;: This returns the count of non-null values in that column.

#### Example Scenario
Suppose we have a table named user_data with columns user_id, email, and gender. The data might look like this:

MARKDOWN
| user_id | email           | gender |
|---------|-----------------|--------|
| 1       | john@example.com| M      |
| 2       | NULL            | F      |
| 3       | jane@example.com|        |

Applying the Queries:

SELECT COUNT(*) FROM user_data; would return 3, as there are three rows.
SELECT COUNT(email) FROM user_data; would return 2, since only two emails have data (ignoring NULL values).
SELECT COUNT(gender) FROM user_data; would also return 2, as the third row has no gender specified.

Choosing Between Them:

Use COUNT(*) when you need to know the total number of records in a table.
Use COUNT(column_name) when you specifically want to count only those rows where that column has valid (non-null) data, which is crucial for analyzing specific features.

In summary, COUNT(*) gives an overall row count, while COUNT(column) provides a count of non-null values in that specific column. The choice depends on the requirement: if you're looking at the total number of records or when filtering based on valid entries in a particular column.

