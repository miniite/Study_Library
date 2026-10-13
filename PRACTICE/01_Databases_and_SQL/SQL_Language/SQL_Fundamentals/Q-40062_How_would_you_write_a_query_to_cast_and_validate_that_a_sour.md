---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a query to cast and validate that a source string column of amounts contains only valid numbers before loading?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Data Validation Queries](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Data_Validation_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a source string column containing amounts is loaded into your database with valid numeric values, we need to write an SQL query that both casts and validates the data. This involves using functions like TRY_CAST or CAST, along with conditions to check for invalid formats. The core idea here is to filter out any strings that cannot be cast to a numeric type before loading them.

#### Triaging the Question

The question essentially asks how to write an SQL query that not only casts a string column into a numeric data type but also validates whether the conversion was successful. We need to consider potential edge cases and ensure the query is robust enough to handle different input formats gracefully.

#### First Principles Solution
Identify the Source Column: Assume we have a table source_table with a column named amount_column.
Use TRY_CAST or CAST Function: Depending on your SQL dialect, use either TRY_CAST or CAST. TRY_CAST returns NULL if the conversion fails, while CAST throws an error.
Filter Valid Data: Use WHERE clauses to filter out invalid conversions and ensure only valid numeric values are loaded.
SQL Query Example
SQL
-- Assuming MySQL syntax (other DBMS may use TRY_CAST or similar)
SELECT amount_column
FROM source_table
WHERE TRY_CAST(amount_column AS DECIMAL(10, 2)) IS NOT NULL;

This query attempts to cast amount_column as a decimal with up to two digits after the decimal point. If the conversion fails, it returns NULL, which is filtered out by the IS NOT NULL condition.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Common Mistake: Not handling edge cases like empty strings or non-numeric characters.

#### Follow-Up
Discussing performance implications if the table has a large number of rows and considering indexing strategies to speed up the query execution.
Additional Consideration: In some databases, you might need to handle different data types (e.g., INT, FLOAT, etc.) depending on your specific requirements.

#### Example
Input and Output

Let’s assume we have the following sample input:

source_table
amount_column
100.00
25
abc
-98.76

The query will return only rows where amount_column can be successfully cast to a numeric type:

CODE
| amount_column |
|---------------|
| 100.00        |
| 25            |
| -98.76        |
Self-Check

This solution is correct and covers the primary requirements of casting and validating string data in SQL.

**Comparison:**

```
Input (source_table)	Output (Filtered Rows)
100.00	100.00
25	25
abc
-98.76	-98.76
```

**Code:**

```sql
SELECT amount_column
FROM source_table
WHERE TRY_CAST(amount_column AS DECIMAL(10, 2)) IS NOT NULL;
```

**Example input / output:**

| Input (source_table)    | Output (Filtered Rows) |
|-------------------------|-----------------------|
| 100.00                  | 100.00                |
| 25                      | 25                    |
| abc                     |                       |
| -98.76                  | -98.76                |

**Data:**

| source_table |
|---------------|
| amount_column |
| 100.00        |
| 25            |
| abc           |
| -98.76        |

**Flow:**

1. Input Table
2. Cast & Validate Amounts
3. Filtered Valid Rows
4. Output Table

