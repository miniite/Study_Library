---
type: "practice"
difficulty: "Deep"
---

# Your fact-to-dimension SUM double-counts because the dimension is Type-2 and matches multiple versions per fact. How would you constrain the join to the version effective at the fact's event date?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Joining Type 2 Dimensions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Joining_Type_2_Dimensions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve the issue of double-counting in a SQL query where Type-2 dimension values are matched across different versions, we need to ensure that each fact is joined only with its corresponding effective version of the dimension at the time of the fact's event date. This problem typically arises when dealing with slowly changing dimensions (SCDs), specifically type 2 SCDs, which maintain a history by creating new records for changes.

#### Triage

The core challenge here is to correctly join the fact table with the appropriate version of the dimension based on the event date in the fact table. The interviewer is probing your understanding of SCDs, particularly type 2, and how to handle temporal joins efficiently. We will assume that both the fact and dimension tables are timestamped to capture the effective dates.

#### First-Principles Solution

To address this, we can use a window function or a common table expression (CTE) to identify the correct version of the dimension for each fact. Let's break it down step by step:

Identify the Correct Version: For each row in the fact table, find the most recent version of the dimension that is still effective up to and including the event date.
Join with the Fact Table: Join this identified version of the dimension back into the fact table.
Step-by-Step Solution

Create a CTE for Dimension Versions:

SQL
WITH dim_versions AS (
    SELECT
        dim_key,
        effective_date,
        end_date,
        value -- Other dimension attributes
    FROM dimensions
)

Find the Correct Version per Fact Event Date: We can use a window function to find the latest version of the dimension that is still active up to the fact's event date.

SQL
, selected_versions AS (
    SELECT
        f.*,
        dv.value
    FROM facts f
    JOIN dim_versions dv ON f.dim_key = dv.dim_key
      AND f.event_date BETWEEN dv.effective_date AND COALESCE(dv.end_date, '9999-12-31')
    QUALIFY ROW_NUMBER() OVER (PARTITION BY f.key ORDER BY dv.effective_date DESC) = 1
)

Final Output: Now that we have the correct dimension values for each fact, we can perform our aggregation or other operations.

SQL
SELECT
    s.fact_key,
    s.value -- Other aggregated facts
    dv.value AS dim_value -- Use this in your calculations
FROM selected_versions s;

#### Closing Summary
+ Common Mistakes/Follow-Ups

This approach ensures that each fact is matched with the correct version of its dimension, preventing double-counting. Key points include:

Using window functions to dynamically find the most recent effective version.
Ensuring temporal accuracy in joins by considering both start and end dates.

Common pitfalls include:

Misunderstanding the BETWEEN condition for date ranges.
Forgetting to use a window function to uniquely identify the latest valid dimension record per fact.
Common Follow-Ups
How would you handle this if there are multiple facts with the same event date?
What if dimensions have overlapping effective dates, and how do you resolve ties?

**Comparison:**

| Fact Table | Dimension Table |
|------------|----------------|
| fact_key   | dim_key         |
| event_date | effective_date  |
| value      | end_date        |
|            | value           |

**Code:**

```sql
WITH dim_versions AS (
    SELECT
        dim_key,
        effective_date,
        end_date,
        value -- Other dimension attributes
    FROM dimensions
)
, selected_versions AS (
    SELECT
        f.*,
        dv.value
    FROM facts f
    JOIN dim_versions dv ON f.dim_key = dv.dim_key
      AND f.event_date BETWEEN dv.effective_date AND COALESCE(dv.end_date, '9999-12-31')
    QUALIFY ROW_NUMBER() OVER (PARTITION BY f.key ORDER BY dv.effective_date DESC) = 1
)
SELECT
    s.fact_key,
    s.value -- Other aggregated facts
    dv.value AS dim_value -- Use this in your calculations
FROM selected_versions s;
```

**Example input / output:**

```text
WITH dim_versions AS (
    SELECT
        dim_key,
        effective_date,
        end_date,
        value -- Other dimension attributes
    FROM dimensions
)
, selected_versions AS (
    SELECT
        f.*,
        dv.value
    FROM facts f
    JOIN dim_versions dv ON f.dim_key = dv.dim_key
      AND f.event_date BETWEEN dv.effective_date AND COALESCE(dv.end_date, '9999-12-31')
    QUALIFY ROW_NUMBER() OVER (PARTITION BY f.key ORDER BY dv.effective_date DESC) = 1
)
SELECT
    s.fact_key,
    s.value AS fact_value,
    dv.value AS dim_value
FROM selected_versions s;
```

**Data:**

```sql
-- Example Fact Table Schema
CREATE TABLE facts (
    fact_key INT,
    event_date DATE,
    value DECIMAL(10,2)
);

-- Example Dimension Table Schema (Type-2 SCD)
CREATE TABLE dimensions (
    dim_key INT,
    effective_date DATE,
    end_date DATE,
    value VARCHAR(100),
    PRIMARY KEY(dim_key, effective_date)
);
```

**Flow:**

1. Fact Table
2. Filter by Event Date
3. Window Function to SelectLatest Version
4. Join Fact and Dimension
5. Final Aggregation/Calculation

