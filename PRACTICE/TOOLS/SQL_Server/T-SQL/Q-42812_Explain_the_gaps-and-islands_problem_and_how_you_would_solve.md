---
type: "practice"
difficulty: "Deep"
---

# Explain the gaps-and-islands problem and how you would solve it with ROW_NUMBER differencing.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ CROSS APPLY and OUTER APPLY](<../../../../TOOLS/SQL_Server/T-SQL/CROSS_APPLY_and_OUTER_APPLY.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The gaps-and-islands problem is a common data analysis challenge where we need to identify contiguous groups of rows that are separated by one or more "gaps." For example, given a table of events with timestamps, you might want to find periods of activity and inactivity. In T-SQL, this can be elegantly solved using the ROW_NUMBER function combined with differences (LAG). This approach allows us to identify transitions between consecutive rows, which helps us segment contiguous groups.

Let's start by understanding what we need from a high-level perspective:

We have a table of events with some form of unique identifier and possibly timestamps.
Our goal is to group these events into "islands" (contiguous periods) and identify the boundaries ("gaps") between these islands.
Triaging the Problem

The core question here is how to use T-SQL to identify and segment contiguous rows. We need to:

Identify the start of each island.
Track transitions from one event group to another.
Label or mark the boundaries where a new island begins.

We can assume that our input data might look like this:

SQL
CREATE TABLE Events (
    Id INT PRIMARY KEY,
    Timestamp DATETIME NOT NULL
);

INSERT INTO Events (Id, Timestamp)
VALUES
(1, '2023-10-01 09:00'),
(2, '2023-10-01 09:05'),
(3, '2023-10-01 09:10'),
(4, '2023-10-01 09:20'),
(5, '2023-10-01 09:30'),
(6, '2023-10-01 10:00'),
(7, '2023-10-01 10:15');

#### First Principles Solution

The key steps involve:

Generating Row Numbers: Assign a row number to each event based on its order.
Calculating Differences: Use LAG to calculate the difference between consecutive rows' numbers and timestamps.
Identifying Boundaries: Mark boundaries where differences indicate a gap.

Here's how we can implement this in T-SQL:

SQL
WITH NumberedEvents AS (
    SELECT
        Id,
        Timestamp,
        ROW_NUMBER() OVER (ORDER BY Timestamp) - ROW_NUMBER() OVER () AS RowDiff
)
SELECT
    Id,
    Timestamp,
    LAG(RowDiff, 1, NULL) OVER (ORDER BY Timestamp) AS PrevRowDiff,
    RowDiff
FROM NumberedEvents;

This query generates a RowDiff column that helps us identify transitions between contiguous rows. By comparing the current and previous values of RowDiff, we can detect where gaps occur.

Full Solution with T-SQL

To fully segment the islands, we need to use these differences to mark the boundaries:

SQL
WITH NumberedEvents AS (
    SELECT
        Id,
        Timestamp,
        ROW_NUMBER() OVER (ORDER BY Timestamp) - ROW_NUMBER() OVER () AS RowDiff
)
SELECT
    Id,
    Timestamp,
    CASE
        WHEN PrevRowDiff IS NOT NULL AND PrevRowDiff + 1 = RowDiff THEN 'Gap'
        ELSE 'Island'
    END AS IslandStatus
FROM NumberedEvents
CROSS APPLY (
    SELECT LAG(RowDiff, 1, NULL) OVER (ORDER BY Timestamp)
) T(PrevRowDiff);

This query uses a CASE statement to label the rows based on whether they are part of an island or a gap.

#### Closing Summary
and Common Mistakes

In summary, we identified the gaps-and-islands problem and used ROW_NUMBER combined with LAG to solve it. The key steps involve generating row numbers, calculating differences, and then using these differences to mark boundaries between islands and gaps.

#### Common mistakes
include:

Incorrectly handling NULL values when using LAG.
Overlooking the need for a CROSS APPLY in T-SQL.
Not understanding how ROW_NUMBER works and its limitations.

**Comparison:**

| Id | Timestamp               | IslandStatus |
|----|-------------------------|--------------|
| 1  | 2023-10-01 09:00:00.000 | Island       |
| 2  | 2023-10-01 09:05:00.000 | Island       |
| 3  | 2023-10-01 09:10:00.000 | Island       |
| 4  | 2023-10-01 09:20:00.000 | Gap          |
| 5  | 2023-10-01 09:30:00.000 | Island       |
| 6  | 2023-10-01 10:00:00.000 | Gap          |
| 7  | 2023-10-01 10:15:00.000 | Island       |

**Code:**

```sql
WITH NumberedEvents AS (
    SELECT
        Id,
        Timestamp,
        ROW_NUMBER() OVER (ORDER BY Timestamp) - ROW_NUMBER() OVER () AS RowDiff
)
SELECT
    Id,
    Timestamp,
    CASE
        WHEN PrevRowDiff IS NOT NULL AND PrevRowDiff + 1 = RowDiff THEN 'Gap'
        ELSE 'Island'
    END AS IslandStatus
FROM NumberedEvents
CROSS APPLY (
    SELECT LAG(RowDiff, 1, NULL) OVER (ORDER BY Timestamp)
) T(PrevRowDiff);
```

**Example input / output:**

```text
WITH NumberedEvents AS (
    SELECT
        Id,
        Timestamp,
        ROW_NUMBER() OVER (ORDER BY Timestamp) - ROW_NUMBER() OVER () AS RowDiff
)
SELECT
    Id,
    Timestamp,
    CASE
        WHEN PrevRowDiff IS NOT NULL AND PrevRowDiff + 1 = RowDiff THEN 'Gap'
        ELSE 'Island'
    END AS IslandStatus
FROM NumberedEvents
CROSS APPLY (
    SELECT LAG(RowDiff, 1, NULL) OVER (ORDER BY Timestamp)
) T(PrevRowDiff);
```

**Data:**

```sql
CREATE TABLE Events (
    Id INT PRIMARY KEY,
    Timestamp DATETIME NOT NULL
);

INSERT INTO Events (Id, Timestamp)
VALUES
(1, '2023-10-01 09:00'),
(2, '2023-10-01 09:05'),
(3, '2023-10-01 09:10'),
(4, '2023-10-01 09:20'),
(5, '2023-10-01 09:30'),
(6, '2023-10-01 10:00'),
(7, '2023-10-01 10:15');
```

**Flow:**

1. Start
2. Generate Row Numbers
3. Calculate Differences
4. Identify Boundaries
5. Label Islands and Gaps
6. End

