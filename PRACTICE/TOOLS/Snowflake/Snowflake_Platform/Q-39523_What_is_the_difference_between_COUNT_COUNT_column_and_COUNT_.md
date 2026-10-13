---
type: "practice"
difficulty: "Basic"
---

# What is the difference between COUNT(*), COUNT(column), and COUNT(DISTINCT column) in a Snowflake query?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Snowflake Loading and Pipelines](<../../../../TOOLS/Snowflake/Snowflake_Platform/Snowflake_Loading_and_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Snowflake, COUNT(*), COUNT(column), and COUNT(DISTINCT column) are used to count rows and columns with different criteria. Understanding these differences is crucial when designing queries to accurately retrieve counts based on your data requirements.

#### Triage

The question asks you to explain the difference between three types of counting operations in Snowflake: COUNT(*), COUNT(column), and COUNT(DISTINCT column). The interviewer is probing your understanding of basic SQL concepts, specifically how these functions behave differently when applied to a dataset. You need to ensure that you are clear about what each operation counts and the implications for performance and data integrity.

#### First-Principles Solution
COUNT(*): This function counts all rows in a result set, regardless of null or non-null values.
COUNT(column): This function counts the number of rows where column is not null.
COUNT(DISTINCT column): This function counts the unique values in the specified column, ignoring duplicate entries.
Detailed Explanation:

COUNT(*) is used when you want to count all rows, including those with null values in every column. For example, if a table has 10 rows and one of them contains nulls, COUNT(*) will still count it.

COUNT(column) is useful when you need to know how many non-null entries there are in a specific column. If the same row can have multiple null values across different columns but needs only one non-null value counted for each row, this function would be appropriate.

COUNT(DISTINCT column) is used when you want to count unique values of a specific column. This can be particularly useful in scenarios where duplicates should not be considered the same. For instance, if column contains repeated entries like "apple", and you want to know how many distinct fruits are present, use this function.

#### Common Mistakes/Follow-Ups

Mistake: Assuming that COUNT(column) counts all rows regardless of nulls.

#### Follow-Up
Explain the difference between COUNT(*) and COUNT(column), especially in cases where there might be null values in columns other than the one being counted.

Mistake: Thinking COUNT(DISTINCT column) is equivalent to COUNT(column).

#### Follow-Up
Clarify that COUNT(DISTINCT column) counts only unique non-null entries, and provide an example where this makes a difference (e.g., having multiple rows with the same value in a column).

Mistake: Overlooking performance implications.

#### Follow-Up
Discuss how these functions can impact query performance depending on the size of your dataset. For instance, COUNT(DISTINCT column) is generally more resource-intensive as it requires additional processing to identify unique values.

#### Closing Summary

Understanding these count operations in Snowflake is essential for writing efficient and accurate queries. Each function serves a specific purpose: COUNT(*) counts all rows, COUNT(column) counts non-null entries in a specific column, and COUNT(DISTINCT column) counts unique values of that column. Awareness of their differences can help you choose the right function based on your data and requirements.

**Code:**

```sql
-- Example usage:
SELECT
    COUNT(*) AS total_rows,
    COUNT(column_name) AS non_null_column_entries,
    COUNT(DISTINCT column_name) AS unique_column_values
FROM table_name;
```

