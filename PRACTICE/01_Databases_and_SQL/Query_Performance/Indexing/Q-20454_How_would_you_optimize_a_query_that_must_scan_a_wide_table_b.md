---
type: "practice"
difficulty: "Intermediate"
---

# How would you optimize a query that must scan a wide table but only three columns, using columnar projection, and quantify the bytes-scanned savings?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ How Indexes Are Used](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/How_Indexes_Are_Used/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To optimize a query that scans a wide table but requires data from only three specific columns, we can leverage columnar storage and indexing to reduce the amount of data read from disk. The goal is to minimize bytes scanned by projecting the required columns early in the process. Let's break down the solution step-by-step.

First, we need to understand that when scanning a wide table, most databases will load an entire row into memory, even if only a few columns are needed. This can be inefficient, especially on large tables or with a high selectivity of data. By using columnar storage and projection, we can significantly reduce the amount of data read.

#### Triage

The question asks us to optimize a query that must scan a wide table but only requires three specific columns. The key aspects are:

Columnar Storage: Store data in columns rather than rows.
Projection: Only include the necessary columns in the result set.
Quantify Savings: Measure the reduction in bytes scanned.

#### Assumptions
The database uses columnar storage, such as Parquet or ORC files.
We can manipulate the query to enforce projection.
The table is large and has many more columns than just the three needed.
Step-by-Step Solution
1. Understand the Data Layout

Assume we have a wide table with 100 columns, but only three are frequently accessed in queries. If this data is stored in row format, scanning the entire table will be inefficient because most of the columns won’t be used.

2. Enforce Columnar Storage

Using columnar storage formats like Parquet ensures that each column's data is stored contiguously. This allows us to read only the required columns, significantly reducing the bytes scanned. For instance, if we have a table with 100 million rows and three specific columns are accessed, in row format, we would scan around 3 billion bytes (assuming 4 bytes per value). With columnar storage, we only read the necessary data.

3. Implement Column Projection

To enforce column projection, modify the SQL query to explicitly select only the required columns:

SQL
SELECT col1, col2, col3 FROM wide_table WHERE condition;

This tells the database management system (DBMS) to skip loading other columns, thus reducing the total bytes read.

4. Measure Bytes Scanned Savings

To quantify the savings, we need to compare the actual bytes scanned before and after applying columnar projection.

Before Projection: The DBMS loads an entire row for each selected row.
After Projection: Only the necessary columns are loaded.

Assume:

Each row has 100 columns.
Each column is of type integer, taking up 4 bytes.
There are 1 million rows in the table.

Before Columnar Projection:

PLAINTEXT
Bytes Scanned = Number of Rows * Columns per Row * Bytes per Column
= 1,000,000 * 100 * 4
= 4,000,000,000 bytes (4 GB)

After Columnar Projection:

PLAINTEXT
Bytes Scanned = Number of Rows * Columns Required * Bytes per Column
= 1,000,000 * 3 * 4
= 12,000,000 bytes (12 MB)

Thus, the savings in bytes scanned are:

PLAINTEXT
Savings = Before - After
= 4 GB - 12 MB
= 3.8796875 GB

#### Closing Summary
and Common Mistakes/Follow-Ups

By using columnar storage and enforcing column projection, we can significantly reduce the bytes scanned, leading to faster query execution times. The savings in this example are substantial (over 97%). However, consider edge cases like null values or sparse data that might affect the exact savings.

#### Common mistakes
include:

Failing to understand the importance of columnar storage and its benefits.
Ignoring the overhead of projection when dealing with small datasets or infrequent queries.
Not considering other optimization techniques such as indexing on frequently accessed columns.

For follow-ups, consider discussing the trade-offs between different storage formats (e.g., row vs. column), the impact of data compression, and the implications for write operations in a columnar database.

**Comparison:**

| Before Projection | After Projection |
|------------------|-----------------|
| Bytes Scanned: 4 GB | Bytes Scanned: 12 MB |
| Rows: 1,000,000   | Columns Selected: col1, col2, col3 |

**Code:**

```sql
-- SQL query with projection
SELECT col1, col2, col3 FROM wide_table WHERE condition;
```

**Example input / output:**

```text
Input: SELECT col1, col2, col3 FROM wide_table WHERE condition;

Output:
Bytes Scanned Before Projection: 4 GB
Bytes Scanned After Projection: 12 MB
Savings: 3.8796875 GB
```

**Data:**

```plaintext
# Example schema
table_name: wide_table
columns:
- col1 (integer)
- col2 (integer)
- ...
- col98 (integer)
- col99 (integer)
- col100 (integer)
rows: 1,000,000
```

**Flow:**

1. Yes
2. No
3. Query Execution
4. Is Columnar Storage Enabled?
5. Use Column Projection toReduce Bytes Scanned
6. Load Full Rows
7. Calculate Savings (4 GB - 12MB)
8. Estimate Standard BytesScanned

