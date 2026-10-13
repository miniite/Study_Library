---
type: "practice"
difficulty: "Basic"
---

# How would you write a query to verify a column is unique (no duplicate keys) before promoting a staging table to the warehouse?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Deduplication in SQL](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Deduplication_in_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To verify if a column in a staging table contains any duplicates, we can use SQL to check for uniqueness. This involves writing a query that identifies if there are any duplicate keys in the specified column and ensuring no row has more than one occurrence of the same key. Here’s how you would approach this:

#### Triage

The question is asking us to write an SQL query to ensure that a particular column in a staging table does not contain any duplicates before promoting it to the main warehouse. The core task involves identifying if there are duplicate values in the specified column.

#### First-Principles Solution
Understand the Requirements: We need to find out if the given column has unique values or if there are any duplicate entries.
Use a Common Table Expression (CTE): This can help us break down the problem into smaller, manageable parts and make the query more readable.
Compare the Count of Unique Values with the Total Count: If they differ, it means there are duplicates.
SQL Query
SQL
WITH unique_check AS (
    SELECT COUNT(DISTINCT column_name) AS unique_count,
           COUNT(*) AS total_count
    FROM staging_table
)
SELECT
    CASE
        WHEN unique_count = total_count THEN 'Unique'
        ELSE 'Non-Unique'
    END AS uniqueness_status;

#### Closing Summary
and Common Mistakes/Follow-Ups

This query uses a CTE to first calculate the count of distinct values (unique_count) and then compares it with the total number of rows in the table. If unique_count equals total_count, it means all values are unique, otherwise, there are duplicates.

#### Common Mistakes

Forgetting to use DISTINCT when counting unique values.
Not checking the relationship between unique_count and total_count.
Using complex conditions without a clear understanding of what uniqueness implies.
Self-Check

This query correctly checks for unique column values in SQL, ensuring no duplicates exist before promoting the staging table. It is both correct and comprehensive.

