---
type: "practice"
difficulty: "Intermediate"
---

# How would you assign a version number per key so the newest row gets version 1 (for SCD-style processing)?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ ROW_NUMBER RANK and DENSE_RANK](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/ROW_NUMBER_RANK_and_DENSE_RANK.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to understand that we are essentially performing Slowly Changing Dimension (SCD) Type 2 style processing in SQL. The key is to assign a version number based on the timestamp of each row, with the newest entry getting a version number of 1. This involves using window functions to rank rows by their timestamps and then assigning the appropriate version numbers.

#### Triage

The question asks for a method to generate a version number per key such that the most recent record gets the highest (usually 1) version number. The core idea is leveraging SQL's window functions, particularly ROW_NUMBER() or similar ranking mechanisms.

#### First-Principles Solution
Identify the Key Columns: Determine which columns define the uniqueness of your records.
Rank Rows by Timestamp: Use a window function to rank rows based on their timestamp, with the most recent row getting a lower rank number (e.g., 1).
Assign Version Numbers: Map the ranked results to version numbers such that the highest rank gets 1.

Let's assume we have a table named customer_history with columns customer_id, effective_date, and details. The goal is to add a version_number column.

SQL
WITH RankedRecords AS (
    SELECT
        customer_id,
        effective_date,
        details,
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY effective_date DESC) as rn
    FROM
        customer_history
)
SELECT
    customer_id,
    effective_date,
    details,
    CASE
        WHEN rn = 1 THEN 1
        ELSE 0
    END AS version_number
INTO
    customer_history_versioned
FROM
    RankedRecords;

#### Closing Summary
and Common Mistakes/Follow-Ups

This approach ensures that the most recent record for each customer_id gets a version_number of 1, while all older records get 0. Here are some common pitfalls to avoid:

Incorrect Window Partitioning: Ensure you partition by the correct key(s) (e.g., customer_id in our example).
Timestamp Order Direction: Make sure to order timestamps correctly (DESC for most recent first, ASC if you prefer earliest first).
Version Number Mapping: Using a CASE statement effectively maps the rank to the desired version number.

If asked further questions, discuss potential optimizations (e.g., indexing on the partitioning key), handling null or duplicate values in the timestamp column, and the implications of this approach in a larger ETL pipeline.

**Comparison:**

| customer_id | effective_date | details          | version_number |
|-------------|-----------------|------------------|---------------|
| A           | 2024-09-30      | Newer Details    | 1             |
| B           | 2023-11-15      | Initial Details  | 1             |

**Code:**

```sql
WITH RankedRecords AS (
    SELECT
        customer_id,
        effective_date,
        details,
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY effective_date DESC) as rn
    FROM
        customer_history
)
SELECT
    customer_id,
    effective_date,
    details,
    CASE
        WHEN rn = 1 THEN 1
        ELSE 0
    END AS version_number
INTO
    customer_history_versioned
FROM
    RankedRecords;
```

**Example input / output:**

```text
Input:
WITH RankedRecords AS (
    SELECT
        customer_id,
        effective_date,
        details,
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY effective_date DESC) as rn
    FROM
        customer_history
)
SELECT
    customer_id,
    effective_date,
    details,
    CASE
        WHEN rn = 1 THEN 1
        ELSE 0
    END AS version_number
INTO
    customer_history_versioned
FROM
    RankedRecords;

Output:
| customer_id | effective_date | details          | version_number |
|-------------|-----------------|------------------|---------------|
| A           | 2024-09-30      | Newer Details    | 1             |
| B           | 2023-11-15      | Initial Details  | 1             |
```

**Data:**

```sql
customer_history
| customer_id | effective_date | details          |
|-------------|-----------------|------------------|
| A           | 2023-10-01      | Old Details      |
| A           | 2024-09-30      | Newer Details    |
| B           | 2023-11-15      | Initial Details  |

customer_history_versioned
| customer_id | effective_date | details          | version_number |
|-------------|-----------------|------------------|---------------|
| A           | 2024-09-30      | Newer Details    | 1             |
| B           | 2023-11-15      | Initial Details  | 1             |
```

**Flow:**

1. rn=1
2. rn>1
3. Input Table
4. Ranked Records CTE
5. Row Number
6. Assign version_number = 1
7. Assign version_number = 0
8. Versioned Output Table

