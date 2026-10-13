---
type: "practice"
difficulty: "Basic"
---

# What is a temporary / scratch table in an ETL step and when would you use one instead of a CTE?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Schema Migration Risks](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Schema_Migration_Risks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

A temporary or scratch table in an ETL (Extract, Transform, Load) process is a database object that exists only during the current session or until explicitly dropped. It's often used to store intermediate results and can be more flexible than Common Table Expressions (CTEs). You might choose to use temporary tables over CTEs when dealing with large datasets or complex transformations that are better suited for table storage.

#### Triage

The question is asking about the differences between using a temporary table versus a CTE in an ETL process. The interviewer is likely probing your understanding of both concepts and their practical applications, as well as your ability to choose the right tool for the job based on specific requirements.

#### First-Principles Solution
Temporary Tables:
Definition: A temporary table is created within a database session and is automatically dropped when the session ends.
Use Case: Ideal for intermediate results that are needed across multiple steps in an ETL process or where large datasets need to be processed.
Common Table Expressions (CTEs):
Definition: CTEs provide a way to temporarily define a table within a query, which can then be used later in the same query.
Use Case: Suitable for complex queries that can be broken down into simpler parts and executed together without persisting data.
Scenario Analysis:

When to Use Temporary Tables:

Large Datasets: If your transformation step involves processing large datasets, a temporary table allows you to store intermediate results efficiently.
Complex Transformations: When the transformation logic is complex or needs to be reused in multiple steps of an ETL process.

When to Use CTEs:

Simple Query Breakdown: If your query can be simplified by breaking it down into smaller, reusable parts.
Readability and Maintainability: For queries that need to be broken down for better readability or maintenance.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

Temporary tables are more suitable when dealing with large datasets or complex transformations that span multiple steps of an ETL process. They offer the advantage of persisting intermediate results, making them easier to manage and reuse. On the other hand, CTEs are better for simpler queries where readability and performance are crucial.

#### Common mistakes
include:

Overusing temporary tables when a simple query can be broken down using CTEs.
Not considering the storage requirements when deciding between temporary tables and CTEs.
Common Follow-Ups:
"Can you walk me through an example of how to use both a temporary table and a CTE in an ETL process?"
"What are some performance considerations when choosing between these two approaches?"

**Comparison:**

```
Table Example:
column1	transformed_column
1	DATA1
2	DATA2
```

**Code:**

```

-- Example using Temporary Table CREATE TEMPORARY TABLE intermediate_results AS SELECT column1, transformed_column FROM raw_data;

INSERT INTO final_table (column1, transformed_column) SELECT * FROM intermediate_results;

CODE
Copy

```sql
-- Example using CTE
WITH transformed_data AS (
    SELECT column1, transformed_column
    FROM raw_data
)
SELECT * FROM transformed_data;
```

**Example input / output:**

```
Input:
CREATE TABLE raw_data (column1 INT, column2 TEXT);
INSERT INTO raw_data VALUES (1, 'Data1'), (2, 'Data2');
Output with Temporary Table:
SQL
Copy
-- Intermediate Results
CREATE TEMPORARY TABLE intermediate_results AS
SELECT column1, upper(column2) as transformed_column
FROM raw_data;

-- Final Table Insertion
INSERT INTO final_table (column1, transformed_column)
SELECT * FROM intermediate_results;
final_table: (column1 INT, transformed_column TEXT) with values: (1, DATA1), (2, DATA2).
Output with CTE:
SQL
Copy
-- Transformed Data
WITH transformed_data AS (
    SELECT column1, upper(column2) as transformed_column
    FROM raw_data
)
SELECT * FROM transformed_data;
transformed_data: (column1 INT, transformed_column TEXT) with values: (1, DATA1), (2, DATA2).
```

**Data:**

```
Schema:
raw_data: Contains the raw data to be transformed.
final_table: The table where the final results will be stored.
Input -> Output Example:

Input:

SQL
Copy
CREATE TABLE raw_data (column1 INT, column2 TEXT);
INSERT INTO raw_data VALUES (1, 'Data1'), (2, 'Data2');

Transformation with Temporary Table:

SQL
Copy
CREATE TEMPORARY TABLE intermediate_results AS
SELECT column1, upper(column2) as transformed_column
FROM raw_data;

INSERT INTO final_table (column1, transformed_column)
SELECT * FROM intermediate_results;

Transformation with CTE:

SQL
Copy
WITH transformed_data AS (
    SELECT column1, upper(column2) as transformed_column
    FROM raw_data
)
SELECT * FROM transformed_data;
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start ETL Process
6. Use Temporary Table?
7. Create and PopulateIntermediate Results
8. Insert into Final Table
9. Use CTE?
10. Break Down Query into CTESteps
11. Execute CTE-Based Queries
12. Directly Write Queries WithoutTemporary Tables or CTEs

