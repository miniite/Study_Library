---
type: "practice"
difficulty: "Intermediate"
---

# How would you remove duplicate rows keeping only the first occurrence using ROW_NUMBER()?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ ROW_NUMBER RANK and DENSE_RANK](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/ROW_NUMBER_RANK_and_DENSE_RANK.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To remove duplicate rows and keep only the first occurrence, we can leverage SQL window functions, specifically ROW_NUMBER(). This approach involves assigning a unique number to each row within each partition of the data based on certain criteria. By filtering out rows where ROW_NUMBER() is greater than 1, we effectively eliminate duplicates.

#### Triaging the Question

The core of this question revolves around understanding how to use SQL window functions to manipulate and filter dataset duplicates efficiently. The interviewer might be probing your knowledge of:

SQL window functions, particularly ROW_NUMBER()
Data partitioning and ordering
Query performance considerations

#### Assumptions
We are working with a relational database that supports window functions.
The table has an identifier (e.g., id) and other columns we want to keep.

#### Solution
from First Principles

Here's how you can achieve this:

Partition the Data: Group rows based on the criteria for identifying duplicates.
Assign Row Numbers: Use ROW_NUMBER() within each partition to number the rows.
Filter Rows: Retain only those rows where ROW_NUMBER() is 1, which means they are the first occurrence in their respective partitions.

#### Example
SQL Query
SQL
WITH RankedData AS (
    SELECT id, column2, column3, ROW_NUMBER() OVER (PARTITION BY id ORDER BY id) as rn
    FROM YourTable
)
SELECT id, column2, column3
FROM RankedData
WHERE rn = 1;

#### Closing Summary
and Common Mistakes/Follow-Ups

This query ensures that only the first occurrence of each id is retained. The use of a common table expression (CTE) helps in breaking down the problem into smaller, more manageable parts.

Common Pitfalls
Incorrect Partitioning: Failing to properly define the partition criteria can lead to incorrect results.
Ordering Clause: Not specifying an ordering clause within ROW_NUMBER() might result in arbitrary duplicates being retained.
Performance Issues: If not optimized, large datasets can experience significant performance degradation.
Closing Thoughts

This approach is efficient and straightforward but requires careful consideration of the data characteristics. Always validate with sample data to ensure correctness.

**Code:**

```sql
WITH RankedData AS (
    SELECT id, column2, column3, ROW_NUMBER() OVER (PARTITION BY id ORDER BY id) as rn
    FROM YourTable
)
SELECT id, column2, column3
FROM RankedData
WHERE rn = 1;
```

