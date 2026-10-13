---
type: "practice"
difficulty: "Deep"
---

# Explain how Delta Lake data skipping uses min/max column statistics in the transaction log to prune Parquet files without any index.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Min Max Statistics and Data Skipping](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Min_Max_Statistics_and_Data_Skipping.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Delta Lake's data skipping feature is a powerful optimization technique that allows for efficient query execution by leveraging metadata stored in the transaction log. This method prunes unneeded Parquet files based on min/max column statistics, reducing the number of files scanned and improving performance without requiring an explicit index.

To understand how Delta Lake achieves this, let’s break down the process from first principles:

Transaction Log: The Delta Lake transaction log records all changes to the dataset in a sequence of commit messages.
Min/Max Column Statistics: These statistics are generated for each column during write operations and stored in the metadata of Parquet files.

The goal is to use these min/max values to determine which Parquet files can be skipped when querying specific columns. This requires understanding how Delta Lake organizes data, how it writes changes, and how query execution leverages this information.

Let’s dive into the details:

1. Transaction Log and Metadata

Delta Lake's transaction log is a sequence of JSON objects, each representing a commit to the dataset. Each commit includes metadata about the operations performed (e.g., inserts, deletes). This metadata helps in determining the changes made to the data over time.

2. Min/Max Column Statistics

When data is written to Parquet files in Delta Lake, min/max statistics are computed for each column and stored within the Parquet metadata. These statistics provide a quick way to understand the range of values in each column without reading all rows.

3. Query Execution and Pruning

During query execution, Delta Lake uses these min/max statistics from the transaction log to determine which files contain data relevant to the current query. By comparing the query’s conditions with the known min/max ranges, it can skip scanning entire Parquet files that do not intersect with the query range.

Step-by-Step Process:
Query Analysis: The SQL parser breaks down the query into its constituent parts and identifies which columns are involved.
Metadata Retrieval: For each column in the query, Delta Lake retrieves the min/max statistics from the transaction log.
Range Comparison: These statistics are compared with the query conditions to determine if any Parquet files can be skipped.
File Selection: Based on the results of this comparison, Delta Lake selects only those Parquet files that fall within the relevant range.

#### Example

Suppose we have a table sales in Delta Lake and run a query:

SQL
SELECT * FROM sales WHERE price > 100 AND year = 2023;

Delta Lake retrieves min/max statistics for columns price and year from the transaction log. If year is set to 2023, it will skip Parquet files where the majority of records are from other years.

#### Common Mistakes
Misunderstanding Statistics: Not fully comprehending how min/max values are used in pruning can lead to incorrect assumptions about query performance.
Ignoring Metadata: Failing to leverage transaction log metadata can result in suboptimal query plans that unnecessarily scan more files than necessary.

#### Closing Summary

Delta Lake’s data skipping feature effectively optimizes query execution by using min/max column statistics and the transaction log to prune unneeded Parquet files, thereby reducing I/O operations and improving performance. Understanding this process is crucial for both designing efficient Delta Lake workflows and troubleshooting performance issues.

**Comparison:**

| Column | Min Value | Max Value |
|--------|-----------|-----------|
| price  | 50        | 150       |
| year   | 2022      | 2024      |

**Code:**

```python
# Example Python code to retrieve metadata from a Delta table
from delta.tables import DeltaTable

delta_table = DeltaTable("sales")
meta_data = delta_table.history()
```

**Example input / output:**

```text
Input Query:
SELECT * FROM sales WHERE price > 100 AND year = 2023;

Output Result (Dry Run):
- Delta Lake checks min/max stats for columns `price` and `year`.
- Finds that only files with `year=2023` are relevant.
- Skips scanning other years' data, reducing I/O to just the needed files.
```

**Data:**

```json
{
  "column_statistics": {
    "price_min": 50,
    "price_max": 150,
    "year_min": 2022,
    "year_max": 2024
  }
}
```

**Flow:**

1. No
2. Yes
3. Query Analysis
4. Retrieve min/max stats fromtransaction log
5. Compare with queryconditions
6. Are relevant files found?
7. Scan all files
8. Select and scan only relevantfiles

