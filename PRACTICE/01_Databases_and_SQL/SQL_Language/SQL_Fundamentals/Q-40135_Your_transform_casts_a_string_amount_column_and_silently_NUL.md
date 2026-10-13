---
type: "practice"
difficulty: "Intermediate"
---

# Your transform casts a string amount column and silently NULLs bad rows, shrinking a revenue metric. How would you use TRY_CAST/SAFE_CAST to quarantine bad rows into a rejects table instead of dropping them?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Data Validation Queries](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Data_Validation_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem using TRY_CAST or SAFE_CAST, we need to transform our string column into a numeric type, such as an integer or float, while handling any bad data that can't be cast properly by placing it in a separate "rejects" table. This approach avoids silently dropping rows with bad data and provides us with more insight into where the issues lie.

#### Triaging the Question
Objective: Use TRY_CAST or SAFE_CAST to handle invalid string-to-number conversions.
Key Concepts: SQL casting, error handling, table joins, subqueries, and creating a separate "rejects" table.

#### Assumptions
The original dataset has a column with string values that should be numeric but includes some bad data (e.g., non-numeric strings).
Step-by-Step Solution

Identify the Column to Transform:

Let's assume we have a revenue table with a column named amount.

TRY_CAST or SAFE_CAST Usage:

We will use TRY_CAST (or SAFE_CAST, if available in your SQL dialect) to attempt converting the string values to a numeric type.
Rows that cannot be casted will result in NULL and can be identified by checking for these NULL values.

Create a Rejected Table:

We'll create a separate table or use an existing one to store rows with bad data.
This table will capture the original row, the attempted cast value, and any relevant information like the original string value.

Insert Good and Bad Rows:

Insert good rows (those that can be successfully casted) into the main transformed table.
Insert rejected rows into the "rejects" table.

#### First-Principles Solution
SQL
-- Step 1: Create a temporary table to hold rejected data
CREATE TABLE IF NOT EXISTS rejects (
    id INT,
    amount STRING,
    error_message STRING
);

-- Step 2: Try casting the string 'amount' column into an integer or float
WITH transformed_data AS (
    SELECT
        id,
        TRY_CAST(amount AS INT) AS casted_amount,
        CASE
            WHEN TRY_CAST(amount AS INT) IS NULL THEN amount
            ELSE ''
        END AS rejected_amount
    FROM revenue
)

-- Step 3: Insert good and bad rows into their respective tables
INSERT INTO main_table (id, transformed_amount)
SELECT id, casted_amount
FROM transformed_data
WHERE casted_amount IS NOT NULL;

INSERT INTO rejects (id, amount, error_message)
SELECT
    id,
    rejected_amount AS amount,
    'Failed to cast: ' || rejected_amount AS error_message
FROM transformed_data
WHERE casted_amount IS NULL;

#### Closing Summary
and Common Mistakes

#### Closing Summary
By using TRY_CAST or SAFE_CAST, we can handle bad data gracefully by moving it to a separate "rejects" table. This approach ensures that no rows are silently dropped, providing us with more information about where the issues lie.

#### Common Mistakes
Forgetting to create the "rejects" table beforehand.
Not handling NULL values correctly, leading to unexpected behavior or silent failures.
Overlooking the need for logging detailed error messages in the "rejects" table.
Code Block
SQL
-- SQL code block showing the solution
Data Blocks (Optional)
Sample data schema:
JSON
{
  "revenue": [
    {"id": 1, "amount": "100"},
    {"id": 2, "amount": "abc"},
    {"id": 3, "amount": "50"}
  ]
}
Input-Output (Optional)
SQL
-- Example of input and output data
Tables (Optional)
has
revenue
rejects
Flowchart (Optional)
Transform Data
Cast Amount
Insert into Main Table
Create Rejected Rows
Log to Rejects Table

**Comparison:**

```
has
revenue
rejects
```

**Code:**

```sql
-- SQL code block showing the solution
WITH transformed_data AS (
    SELECT
        id,
        TRY_CAST(amount AS INT) AS casted_amount,
        CASE
            WHEN TRY_CAST(amount AS INT) IS NULL THEN amount
            ELSE ''
        END AS rejected_amount
    FROM revenue
)
INSERT INTO main_table (id, transformed_amount)
SELECT id, casted_amount
FROM transformed_data
WHERE casted_amount IS NOT NULL;

INSERT INTO rejects (id, amount, error_message)
SELECT
    id,
    rejected_amount AS amount,
    'Failed to cast: ' || rejected_amount AS error_message
FROM transformed_data
WHERE casted_amount IS NULL;
```

**Example input / output:**

```text
-- Example of input and output data

Input:
revenue =
[
  { id: 1, amount: '100' },
  { id: 2, amount: 'abc' },
  { id: 3, amount: '50' }
]

Output:
main_table =
[
  { id: 1, transformed_amount: 100 },
  { id: 3, transformed_amount: 50 }
]

rejects =
[
  { id: 2, amount: 'abc', error_message: 'Failed to cast: abc' }
]
```

**Data:**

```json
{
  "revenue": [
    {"id": 1, "amount": "100"},
    {"id": 2, "amount": "abc"},
    {"id": 3, "amount": "50"}
  ]
}
```

**Flow:**

1. Transform Data
2. Cast Amount
3. Insert into Main Table
4. Create Rejected Rows
5. Log to Rejects Table

