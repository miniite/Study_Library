---
type: "practice"
difficulty: "Deep"
---

# How does Parquet's internal structure (row groups, column chunks, page-level min/max stats) let engines skip data, and how is that different from a database index?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Min Max Statistics and Data Skipping](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Min_Max_Statistics_and_Data_Skipping.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Parquet’s internal structure allows engines to efficiently skip large portions of data during query execution by leveraging row groups, column chunks, and page-level min/max statistics. This differs significantly from traditional database indexing because it operates at a lower level within the file format itself.

Firstly, let's break down Parquet's internal architecture:

Row Groups: These are contiguous segments of rows in a Parquet file.
Column Chunks: Each column is stored as separate chunks across row groups.
Page-Level Min/Max Stats: Each page (a unit within a column chunk) stores the minimum and maximum values, which help in pruning unnecessary data.

To understand how these components enable efficient skipping:

Row Groups: The engine can read a specific range of rows by determining which row group contains those rows.
Column Chunks: Once a relevant row group is identified, the engine looks at column chunks to narrow down further.
Page-Level Min/Max Stats: These stats help in skipping entire pages that cannot contain the values needed for a query.

#### Triage

The question aims to explore how Parquet's design allows efficient data access and pruning during queries, contrasting it with database indexing mechanisms. The focus is on understanding the underlying mechanics and their benefits over traditional indexes.

Detailed Solution
1. Row Groups

Parquet organizes data into row groups, each containing a fixed number of rows (typically around 64KB to 1MB). This grouping ensures that:

Queries can be executed by only processing relevant row groups.
Small files (row groups) facilitate faster disk I/O operations.
2. Column Chunks

Columns are split into column chunks, which contain multiple row groups but focus on a single column’s values. This allows:

Engines to skip entire columns that do not match the query conditions.
Efficient merging of results from different row groups and their associated column chunks.
3. Page-Level Min/Max Stats

Each page within a column chunk stores min/max statistics, which are crucial for pruning operations:

These stats help in determining if a page contains values relevant to the current query.
Pages that do not match the query range can be skipped entirely, reducing unnecessary processing.
Comparison with Database Indexes

Traditional database indexes work by maintaining a sorted or hashed version of one or more columns. They are used for quick lookups and ensure efficient retrieval of data based on specific conditions. However, they have limitations:

Scanning: Index scans still require scanning the entire index to find matching rows.
Maintenance Overhead: Updating indexes can be resource-intensive.

In contrast, Parquet’s structure allows:

Direct Skips: The engine can directly skip over sections of data that are irrelevant based on query conditions.
Reduced I/O Operations: By leveraging row groups and column chunks, the overall disk I/O required is significantly reduced.

#### Closing Summary

Parquet's design leverages row groups, column chunks, and page-level min/max stats to enable efficient data access and pruning during queries. This approach contrasts with traditional database indexing by providing direct skips over irrelevant data, reducing overall I/O operations and improving query performance.

#### Common Mistakes/Follow-Ups
Misunderstanding Row Group Size: Thinking that row groups are too large for effective skipping.
Ignoring Column Chunks: Failing to recognize the importance of column chunk organization in pruning operations.
Overlooking Page-Level Stats: Not fully appreciating how min/max stats help in skipping pages.

Understanding these components and their interactions is crucial for optimizing Parquet-based queries effectively.

**Comparison:**

```
Row Groups	Columns	Size in Bytes	Column Chunks
1	col1, col2	65536	1, 2
```

**Code:**

```python
# Example function to simulate data access using Parquet's structure

def parquet_query(column, min_val, max_val):
    row_groups = get_row_groups()

    for group in row_groups:
        column_chunks = group.get_column_chunks()

        for chunk in column_chunks:
            pages = chunk.get_pages()

            for page in pages:
                if (page.min_value < min_val or page.max_value > max_val) and is_relevant(column, page):
                    # Process relevant data
                    process_data(page)
```

**Example input / output:**

```text
Input: column="col1", min_val=50, max_val=75

Output:
- Row Group 1 - Column Chunk 1 - Page 3 (process_data(page))
- No further processing needed for other pages or chunks
```

**Data:**

```json
{
  "row_groups": [
    {
      "id": 1,
      "columns": ["col1", "col2"],
      "size_in_bytes": 65536,
      "column_chunks": [
        {"chunk_id": 1, "min_max_stats": {"min": 0, "max": 99}, "pages": [1, 2, 3]},
        {"chunk_id": 2, "min_max_stats": {"min": 100, "max": 199}, "pages": [4, 5]}
      ]
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Read Row Group
4. Is Page Min < min_val || Max >max_val?
5. Skip Page
6. Process Data
7. Next Chunk
8. Repeat for Next Chunk
9. Next Row Group
10. End

