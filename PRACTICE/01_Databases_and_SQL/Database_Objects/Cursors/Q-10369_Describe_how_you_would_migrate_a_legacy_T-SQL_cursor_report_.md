---
type: "practice"
difficulty: "Intermediate"
---

# Describe how you would migrate a legacy T-SQL cursor report to a windowed set-based query and validate row-count and checksum parity before cutover.

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Cursors vs Set-based SQL](<../../../../01_Databases_and_SQL/Database_Objects/Cursors/Cursors_vs_Set-based_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this migration task, we need to convert a legacy T-SQL cursor-based report into a more efficient set-based query using window functions. The goal is not only to transform the logic but also to validate the row-count and checksum parity before and after the cutover. Let's break down the process step by step.

#### Triaging the Question

The question requires us to:

Understand the current logic of the cursor report.
Rewrite it using set-based queries, specifically leveraging window functions for more efficient processing.
Ensure that the new query produces the same results as the original cursor-based report.
Validate the row-count and checksum parity before and after cutover.

#### Assumptions
The legacy T-SQL uses a cursor to process records one by one.
There are existing validation mechanisms in place (likely through checksums or counts) that need to be maintained post-migration.
We can assume the table structure and data distribution remain consistent between the old and new implementations.

#### First-Principles Solution
Step 1: Analyze the Legacy Cursor Logic

First, let's examine the cursor-based logic. Cursors are often used for complex operations where the current row value needs to be compared with a previous or next row. For example:

SQL
DECLARE @CurrentRow INT = -1;
DECLARE @PreviousValue INT;
DECLARE @CurrentValue INT;

DECLARE myCursor CURSOR FOR
SELECT column1, column2 FROM myTable;

OPEN myCursor;
FETCH NEXT FROM myCursor INTO @PreviousValue, @CurrentValue;

WHILE @@FETCH_STATUS = 0
BEGIN
    -- Perform operations based on the comparison of previous and current values
    IF (@CurrentRow > -1 AND @CurrentValue > @PreviousValue)
        -- Do something

    SET @PreviousValue = @CurrentValue;
    FETCH NEXT FROM myCursor INTO @CurrentValue, @PreviousValue;
END;

CLOSE myCursor;
DEALLOCATE myCursor;
Step 2: Convert to Set-Based Query Using Window Functions

To convert this logic into a set-based query using window functions, we need to identify the operations performed within the cursor and replicate them in a more efficient manner. For example:

SQL
WITH ProcessedData AS (
    SELECT column1, column2,
           LAG(column2) OVER (ORDER BY column1) AS PreviousValue,
           LEAD(column2) OVER (ORDER BY column1) AS NextValue
    FROM myTable
)
SELECT *
FROM ProcessedData
WHERE CASE
        WHEN @CurrentRow > -1 AND CurrentValue > PreviousValue THEN 1
        ELSE 0
      END = 1;
Step 3: Validate Row-Count and Checksum Parity

To ensure that the migration is successful, we need to validate both row-count and checksum parity. We can do this by comparing the results of the old cursor-based query with the new set-based query.

Validate Row-Count
SQL
-- Old Query
DECLARE @OldRowCount INT;
SELECT @OldRowCount = COUNT(*)
FROM ( /* Legacy T-SQL Cursor Report */ ) AS OldData;

-- New Query
DECLARE @NewRowCount INT;
SELECT @NewRowCount = COUNT(*)
FROM ( /* New Set-Based Query */ ) AS NewData;

IF @OldRowCount <> @NewRowCount
BEGIN
    PRINT 'Row count mismatch detected!';
END;
Validate Checksum Parity

We can use the CHECKSUM_AGG function to compute a checksum of all relevant columns in both queries and compare them.

SQL
-- Old Query
DECLARE @OldChecksum INT;
SELECT @OldChecksum = CHECKSUM_AGG(column1 + column2)
FROM ( /* Legacy T-SQL Cursor Report */ ) AS OldData;

-- New Query
DECLARE @NewChecksum INT;
SELECT @NewChecksum = CHECKSUM_AGG(column1 + column2)
FROM ( /* New Set-Based Query */ ) AS NewData;

IF @OldChecksum <> @NewChecksum
BEGIN
    PRINT 'Checksum mismatch detected!';
END;

#### Closing Summary
and Common Mistakes/Follow-Ups

By converting the cursor-based logic to a set-based query using window functions, we improve performance and maintain consistency. Validating row-count and checksum parity ensures that no data is lost or altered during migration. Some common mistakes include:

Not thoroughly understanding the legacy logic.
Failing to properly validate both row-count and checksum parity.

For follow-ups, one might explore further optimizations such as partitioning for better performance if the table is large, or using parallel queries where appropriate.

**Comparison:**

```
2023-12-02
2023-12-03
2023-12-04
2023-12-05
2023-12-06
2023-12-07
2023-12-08
2023-12-09
2023-12-10
2023-12-11
2023-12-12
LegacyCursorReport
NewSetBasedQuery
OldRowCount
NewRowCount
OldChecksum
NewChecksum
Legacy Logic
New Set-Based Query
Row-Count Validation
Checksum Parity Validation
Migration Validation
```

**Code:**

```sql
-- Legacy T-SQL Cursor Report
DECLARE @CurrentRow INT = -1;
DECLARE @PreviousValue INT;
DECLARE @CurrentValue INT;

DECLARE myCursor CURSOR FOR
SELECT column1, column2 FROM myTable;

OPEN myCursor;
FETCH NEXT FROM myCursor INTO @PreviousValue, @CurrentValue;

WHILE @@FETCH_STATUS = 0
BEGIN
    -- Perform operations based on the comparison of previous and current values
    IF (@CurrentRow > -1 AND @CurrentValue > @PreviousValue)
        -- Do something

    SET @PreviousValue = @CurrentValue;
    FETCH NEXT FROM myCursor INTO @CurrentValue, @PreviousValue;
END;

CLOSE myCursor;
DEALLOCATE myCursor;

-- New Set-Based Query Using Window Functions
WITH ProcessedData AS (
    SELECT column1, column2,
           LAG(column2) OVER (ORDER BY column1) AS PreviousValue,
           LEAD(column2) OVER (ORDER BY column1) AS NextValue
    FROM myTable
)
SELECT *
FROM ProcessedData
WHERE CASE
        WHEN @CurrentRow > -1 AND CurrentValue > PreviousValue THEN 1
        ELSE 0
      END = 1;

-- Validate Row-Count
DECLARE @OldRowCount INT;
SELECT @OldRowCount = COUNT(*)
FROM ( /* Legacy T-SQL Cursor Report */ ) AS OldData;

DECLARE @NewRowCount INT;
SELECT @NewRowCount = COUNT(*)
FROM ( /* New Set-Based Query */ ) AS NewData;

IF @OldRowCount <> @NewRowCount
BEGIN
    PRINT 'Row count mismatch detected!';
END;

-- Validate Checksum Parity
DECLARE @OldChecksum INT;
SELECT @OldChecksum = CHECKSUM_AGG(column1 + column2)
FROM ( /* Legacy T-SQL Cursor Report */ ) AS OldData;

DECLARE @NewChecksum INT;
SELECT @NewChecksum = CHECKSUM_AGG(column1 + column2)
FROM ( /* New Set-Based Query */ ) AS NewData;

IF @OldChecksum <> @NewChecksum
BEGIN
    PRINT 'Checksum mismatch detected!';
END;
```

**Example input / output:**

```text
-- Legacy T-SQL Cursor Report (Dry Run)
DECLARE @CurrentRow INT = -1;
DECLARE @PreviousValue INT;
DECLARE @CurrentValue INT;

DECLARE myCursor CURSOR FOR
SELECT column1, column2 FROM myTable;

OPEN myCursor;
FETCH NEXT FROM myCursor INTO @PreviousValue, @CurrentValue;

WHILE @@FETCH_STATUS = 0
BEGIN
    -- Perform operations based on the comparison of previous and current values
    IF (@CurrentRow > -1 AND @CurrentValue > @PreviousValue)
        PRINT 'Operation performed for row: ' + CAST(@CurrentValue AS VARCHAR(20));

    SET @PreviousValue = @CurrentValue;
    FETCH NEXT FROM myCursor INTO @CurrentValue, @PreviousValue;
END;

CLOSE myCursor;
DEALLOCATE myCursor;

-- New Set-Based Query Using Window Functions (Dry Run)
WITH ProcessedData AS (
    SELECT column1, column2,
           LAG(column2) OVER (ORDER BY column1) AS PreviousValue,
           LEAD(column2) OVER (ORDER BY column1) AS NextValue
    FROM myTable
)
SELECT *
FROM ProcessedData
WHERE CASE
        WHEN @CurrentRow > -1 AND CurrentValue > PreviousValue THEN 1
        ELSE 0
      END = 1;
```

**Data:**

```json
{
  "myTable": {
    "columns": ["column1", "column2"],
    "data": [
      {"column1": 1, "column2": 5},
      {"column1": 2, "column2": 3},
      {"column1": 3, "column2": 7},
      {"column1": 4, "column2": 8},
      {"column1": 5, "column2": 6}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Analyze Legacy Cursor Logic
4. Convert to Set-Based Query
5. Validate Row-Count
6. Validate Checksum Parity
7. Final Validation Passes?
8. Migrate to Production
9. Debug and Repeat

