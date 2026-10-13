---
type: "practice"
difficulty: "Intermediate"
---

# How would you use a recursive CTE to split a comma-delimited string into rows, and why might a set-based split function be better?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Recursive CTEs and Materialization](<../../../../01_Databases_and_SQL/SQL_Language/CTEs_and_Recursive_Queries/Recursive_CTEs_and_Materialization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question, we'll first provide an overview of using a recursive common table expression (CTE) to split a comma-separated string into rows. We will then discuss when a set-based split function might be more advantageous.

#### Overview
Recursive CTEs are a powerful SQL feature that allow us to perform iterative operations on data until a termination condition is met. For the task of splitting a comma-separated string, we can use a recursive CTE to repeatedly parse and process each substring until no further splits are possible. This approach ensures we handle nested commas correctly and provide an accurate split.

However, it's important to note that while recursive CTEs offer flexibility, they might not be the most efficient method for large datasets or complex string structures. A set-based split function, such as STRING_SPLIT in SQL Server, can often outperform recursive solutions due to its optimized internal implementation and ability to handle parallel processing.

#### Triaging the Question
The question primarily asks how to use a recursive CTE to split a comma-separated string and why a set-based split might be better. The key points are:

Understanding the Problem: We need to break down a single string into multiple rows.

#### Assumptions
The input is a single string with values separated by commas, and we want to process each value individually.

Step-by-Step Solution:

Initial Setup: Define a recursive CTE that initializes with the original string as its seed.
Recursive Step: In each iteration, use a regular expression or substring operation to find the next comma-delimited segment.
Termination Condition: Stop when no more commas are found in the current segment.
Result Collection: Collect all segments into rows.

Here's an example using SQL Server syntax:

SQL
DECLARE @input NVARCHAR(MAX) = 'value1,value2,value3';

WITH SplitCTE (originalString, nextSegment, index)
AS (
    -- Base case: Initialize with the full string and starting index 0
    SELECT
        originalString = @input,
        nextSegment = '',
        index = 0
    UNION ALL
    -- Recursive step: Find the next comma and split the string
    SELECT
        s.originalString,
        SUBSTRING(s.originalString, CHARINDEX(',', s.originalString, s.index) + 1, LEN(s.originalString)) AS nextSegment,
        CHARINDEX(',', s.originalString, s.index + 1)
    FROM SplitCTE s
    WHERE CHARINDEX(',', s.originalString, s.index + 1) > 0
)
-- Final result: Collect all segments into rows
SELECT
    segment = CASE WHEN nextSegment = '' THEN originalString ELSE nextSegment END
FROM SplitCTE;

#### Closing Summary
and Common Mistakes/Follow-Ups:

Summary: Recursive CTEs can be used to split comma-separated strings, but this approach may not always be the most efficient. For large datasets or performance-critical scenarios, consider using set-based operations like STRING_SPLIT.
Mistakes to Avoid:
Ignoring the potential overhead of recursive queries, especially in complex environments.
Failing to handle edge cases such as leading/trailing commas or empty strings.

#### Follow-Up
Questions:

What are some alternative methods for splitting a string?
How might you optimize this query for performance?

**Code:**

```sql
-- Example recursive CTE to split comma-separated values
DECLARE @input NVARCHAR(MAX) = 'value1,value2,value3';

WITH SplitCTE (originalString, nextSegment, index)
AS (
    -- Base case: Initialize with the full string and starting index 0
    SELECT
        originalString = @input,
        nextSegment = '',
        index = 0
    UNION ALL
    -- Recursive step: Find the next comma and split the string
    SELECT
        s.originalString,
        SUBSTRING(s.originalString, CHARINDEX(',', s.originalString, s.index) + 1, LEN(s.originalString)) AS nextSegment,
        CHARINDEX(',', s.originalString, s.index + 1)
    FROM SplitCTE s
    WHERE CHARINDEX(',', s.originalString, s.index + 1) > 0
)
-- Final result: Collect all segments into rows
SELECT
    segment = CASE WHEN nextSegment = '' THEN originalString ELSE nextSegment END
FROM SplitCTE;
```

**Example input / output:**

```text
-- Sample input and expected output
DECLARE @input NVARCHAR(MAX) = 'value1,value2,value3';

WITH SplitCTE (originalString, nextSegment, index)
AS (
    -- Base case: Initialize with the full string and starting index 0
    SELECT
        originalString = @input,
        nextSegment = '',
        index = 0
    UNION ALL
    -- Recursive step: Find the next comma and split the string
    SELECT
        s.originalString,
        SUBSTRING(s.originalString, CHARINDEX(',', s.originalString, s.index) + 1, LEN(s.originalString)) AS nextSegment,
        CHARINDEX(',', s.originalString, s.index + 1)
    FROM SplitCTE s
    WHERE CHARINDEX(',', s.originalString, s.index + 1) > 0
)
-- Final result: Collect all segments into rows
SELECT
    segment = CASE WHEN nextSegment = '' THEN originalString ELSE nextSegment END
FROM SplitCTE;
```

**Flow:**

1. Yes
2. No
3. Input String
4. Find First Comma
5. Is Comma Found?
6. Extract Segment
7. Output Result
8. Update Index & Continue

