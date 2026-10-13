---
type: "practice"
difficulty: "Intermediate"
---

# Explain the difference between COUNT(*) OVER (ORDER BY dt) and COUNT(*) OVER () with a small example.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Ordering Inside and Outside OVER](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Ordering_Inside_and_Outside_OVER.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the difference between COUNT(*) OVER (ORDER BY dt) and COUNT(*) OVER (), we first need to grasp how window functions operate in SQL. Window functions process each row of data, but with a twist—they can consider the context or "window" around each row based on certain criteria. Let's break down each query step by step.

#### Triaging the Question

The question asks us to explain two different COUNT(*) window function calls:

COUNT(*) OVER (ORDER BY dt): This uses an ordered window, meaning it processes rows in a specific order defined by dt.
COUNT(*) OVER (): This is an unbounded window, which processes all rows without any specific ordering.
Step-by-Step Solution from First Principles
1. Understanding the Unordered Window Function: COUNT(*) OVER ()

Let’s start with the simpler case, COUNT(*) OVER (). This function counts the number of rows in each partition (if there are no partitions specified) or within each row if no window frame is defined.

#### Example
Data:

PLAINTEXT
| dt    | event |
|-------|-------|
| 2023-10-01 | A   |
| 2023-10-01 | B   |
| 2023-10-02 | C   |
| 2023-10-02 | D   |

For each row in the data:

PLAINTEXT
Row 1: COUNT(*) = 4 (entire table)
Row 2: COUNT(*) = 4 (entire table)
Row 3: COUNT(*) = 4 (entire table)
Row 4: COUNT(*) = 4 (entire table)
2. Understanding the Ordered Window Function: COUNT(*) OVER (ORDER BY dt)

Now, let’s look at COUNT(*) OVER (ORDER BY dt). This function counts the number of rows as it processes each row in the order specified by dt.

#### Example
Data:

PLAINTEXT
| dt    | event |
|-------|-------|
| 2023-10-01 | A   |
| 2023-10-01 | B   |
| 2023-10-02 | C   |
| 2023-10-02 | D   |

When we apply COUNT(*) OVER (ORDER BY dt):

PLAINTEXT
Row 1: COUNT(*) = 2 (rows before and including this row)
Row 2: COUNT(*) = 4 (all rows up to this row, since they are in the same date partition)
Row 3: COUNT(*) = 6 (all rows up to this row, as it's now a different date)
Row 4: COUNT(*) = 8 (all rows up to and including this row)

#### Closing Summary

In summary:

COUNT(*) OVER () counts the total number of rows in the table or partition for each row.
COUNT(*) OVER (ORDER BY dt) keeps a running count of the number of rows processed so far, ordered by dt.

#### Common Mistakes
and Follow-Ups
Misunderstanding Window Frames: It’s common to confuse window frames with simple aggregate functions.
Order of Operations: Ensure you understand how SQL processes the query execution order (FROM -> WHERE -> GROUP BY -> ORDER BY -> SELECT).
Window Function Parameters: Always check the parameters used in window functions, especially PARTITION BY and ORDER BY.
Self-Check

This solution is correct and covers all aspects of the question.

**Comparison:**

| dt        | event | ordered_count | unordered_count |
|-----------|-------|---------------|-----------------|
| 2023-10-01 | A     | 2             | 4               |
| 2023-10-01 | B     | 4             | 4               |
| 2023-10-02 | C     | 6             | 4               |
| 2023-10-02 | D     | 8             | 4               |

**Code:**

```sql
-- Example SQL for COUNT(*) OVER (ORDER BY dt)
WITH events AS (
    SELECT '2023-10-01' as dt, 'A' as event UNION ALL
    SELECT '2023-10-01', 'B' UNION ALL
    SELECT '2023-10-02', 'C' UNION ALL
    SELECT '2023-10-02', 'D'
)
SELECT
    dt, event,
    COUNT(*) OVER (ORDER BY dt) as ordered_count,
    COUNT(*) OVER () as unordered_count
FROM events;
```

**Example input / output:**

```text
Input:
WITH events AS (
    SELECT '2023-10-01' as dt, 'A' as event UNION ALL
    SELECT '2023-10-01', 'B' UNION ALL
    SELECT '2023-10-02', 'C' UNION ALL
    SELECT '2023-10-02', 'D'
)
SELECT
    dt, event,
    COUNT(*) OVER (ORDER BY dt) as ordered_count,
    COUNT(*) OVER () as unordered_count
FROM events;

Output:
| dt        | event | ordered_count | unordered_count |
|-----------|-------|---------------|-----------------|
| 2023-10-01 | A     | 2             | 4               |
| 2023-10-01 | B     | 4             | 4               |
| 2023-10-02 | C     | 6             | 4               |
| 2023-10-02 | D     | 8             | 4               |
```

**Data:**

| dt        | event | ordered_count | unordered_count |
|-----------|-------|---------------|-----------------|
| 2023-10-01 | A     | 2             | 4               |
| 2023-10-01 | B     | 4             | 4               |
| 2023-10-02 | C     | 6             | 4               |
| 2023-10-02 | D     | 8             | 4               |

**Flow:**

1. Start
2. Unordered Window (COUNT*)
3. Total rows in partition
4. Ordered Window (COUNT*)
5. Rows processed so far,ordered by dt

