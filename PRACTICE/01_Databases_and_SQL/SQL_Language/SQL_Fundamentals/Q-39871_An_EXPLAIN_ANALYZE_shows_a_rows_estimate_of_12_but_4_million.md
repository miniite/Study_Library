---
type: "practice"
difficulty: "Deep"
---

# An EXPLAIN ANALYZE shows a rows estimate of 12 but 4 million actual on a join. What causes such a misestimate and which statistics or extended-stats objects would you create to correct it?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Reading EXPLAIN](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Reading_EXPLAIN.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When an EXPLAIN ANALYZE shows a rows estimate of 12 but actually returns 4 million rows, this indicates a significant misestimate in the query optimizer's predictions. The optimizer relies on statistics to make decisions about how to execute queries efficiently, and when these statistics are inaccurate or outdated, it can lead to suboptimal plans.

To understand why such a misestimate occurs, we need to consider several factors:

Data Distribution: If the data has an unexpected distribution that doesn't match historical patterns.
Sampling Errors: The sample size used for generating statistics might be too small or not representative of the entire dataset.
Index Usage: Inadequate indexing can lead to poor estimates, as the optimizer may overestimate the number of rows that will be returned when an index is not available.

To address this issue and improve query optimization, we need to create and maintain accurate statistics on the relevant columns involved in the join operation. Here’s a step-by-step approach:

Identify the Columns Involved: Determine which columns are causing the misestimate by examining the EXPLAIN ANALYZE output.
Create Statistics Objects:
Base Table Statistics: Ensure that detailed statistics for all relevant columns in base tables are up-to-date.
Column-specific Statistics: Create column-specific histogram objects to capture the distribution of values more accurately.

For instance, if the join is based on a column id in both tables, we would create histograms and possibly even bitmap indexes to improve cardinality estimates.

Detailed Steps

Analyze Data Distribution:

Run queries like SELECT COUNT(*) FROM table WHERE id = value; for different ranges of values to understand the distribution.
Use ANALYZE TABLE table_name; to update statistics on individual tables.

Create Histograms:

For columns with a large number of distinct values, create histograms using the following SQL command:
SQL
CREATE STATISTICS histogram_name ON column_name IN TABLE table_name;

#### Example
CREATE STATISTICS id_histogram ON id IN TABLE table1;

Create Bitmap Indexes:

If the join column has many distinct values, consider creating a bitmap index to improve cardinality estimation:
SQL
CREATE BITMAP INDEX idx_name ON table_name (column_name);

Verify Changes:

After making these changes, re-run EXPLAIN ANALYZE to see if the row estimates have improved.
Monitor query performance and make adjustments as needed.

#### Common Mistakes
Over-reliance on Automatic Statistics: Relying solely on automatically generated statistics can lead to inaccuracies. It’s important to manually create and maintain them when necessary.
Ignoring Data Skew: Failing to account for data skew in the tables can cause significant misestimates, especially if certain values are underrepresented or overrepresented.

By following these steps and continuously monitoring query performance, you can significantly reduce the likelihood of such misestimates occurring in your database environment.

