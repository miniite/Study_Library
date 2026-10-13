---
type: "practice"
difficulty: "Intermediate"
---

# How would you enforce a data-quality check that flags any partition where ROW_NUMBER()=1 count doesn't equal the distinct-key count, and why does that catch dedup bugs?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Deterministic Deduplication](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Deterministic_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure data quality in your dataset, one common issue is duplicate records. To detect these, we can use window functions to flag any partition where ROW_NUMBER()=1 count doesn't equal the distinct-key count. This approach effectively catches deduplication issues because if there are duplicates, the ROW_NUMBER() function will assign a row number to each unique key within the same partition, and the count of these rows won't match the distinct keys.

First, let's break down the problem: you need to identify partitions (e.g., by date or user ID) where the count of first-row instances (ROW_NUMBER()=1) is not equal to the number of distinct values. This discrepancy indicates potential duplicates within that partition.

Now, I'll walk through a step-by-step solution from first principles:

#### Triaging the Question
Question: How do we ensure data quality by detecting partitions where ROW_NUMBER() count doesn't match the distinct key count?
What's Probed: Understanding of window functions in SQL, knowledge of partitioning and row numbering, ability to write complex queries.

#### Assumptions
You have a table with columns such as id, partition_key (e.g., date or user ID).
The goal is to identify partitions where the count of distinct keys does not match the count of first-row instances.

#### First Principles Solution
Identify Unique Keys: Use DISTINCT to get a list of unique keys within each partition.
Assign Row Numbers: Use the ROW_NUMBER() window function to assign a row number to each instance in these partitions, starting from 1.
Count Rows with Row Number 1: Count how many rows have a row number of 1 for each partition.
Compare Counts: Compare the counts from steps 2 and 3; if they are different, it indicates potential duplicates.
SQL Query

Here’s a sample query that implements this approach:

SQL
WITH unique_keys AS (
    SELECT
        partition_key,
        id,
        ROW_NUMBER() OVER (PARTITION BY partition_key, id ORDER BY some_column) as row_num
    FROM
        your_table
)
SELECT
    partition_key,
    COUNT(DISTINCT id) as distinct_count,
    SUM(CASE WHEN row_num = 1 THEN 1 ELSE 0 END) as first_row_count
FROM
    unique_keys
GROUP BY
    partition_key
HAVING
    COUNT(DISTINCT id) <> SUM(CASE WHEN row_num = 1 THEN 1 ELSE 0 END);
Explanation of the Query
CTE (Common Table Expression): unique_keys uses a window function to assign a row number for each unique key within the partition.
Main Query: For each partition, it counts distinct keys and sums up rows where the row number is 1. If these two counts differ, it flags that partition as having potential duplicates.

#### Closing Summary

The ROW_NUMBER() approach effectively identifies partitions with duplicates by comparing the count of unique keys to the count of first-row instances within those partitions. This method ensures robust data quality checks and helps in maintaining the integrity of your dataset.

#### Common Mistakes/Follow-ups
Not Considering Order: Ensure that you order correctly when using ROW_NUMBER() to avoid false positives.
Ignoring Nulls/Empty Values: Handle null or empty values appropriately, as they can affect row counts.

#### Performance Considerations
For very large datasets, performance optimization might be necessary. Indexes on the partition key and ordering column can improve efficiency.

