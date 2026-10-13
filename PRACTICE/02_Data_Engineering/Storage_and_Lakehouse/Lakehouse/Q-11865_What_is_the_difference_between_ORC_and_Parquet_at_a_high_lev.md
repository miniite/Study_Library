---
type: "practice"
difficulty: "Basic"
---

# What is the difference between ORC and Parquet at a high level?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Columnar Formats in the Lake](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Columnar_Formats_in_the_Lake.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

ORC (Optimized Row Column) and Parquet are both columnar storage formats used in data lakes, but they serve different purposes and have distinct characteristics. To understand their differences, we need to consider how each format is designed to handle large-scale data processing and analytics.

Firstly, let's summarize the core difference: ORC is optimized for transactional workloads where you might write data frequently, while Parquet excels in scenarios with frequent reads but infrequent writes. This distinction stems from their design philosophies and trade-offs.

Now, let’s dive deeper:

Design Philosophy: ORC was developed by Cloudera as an improvement over SequenceFile for Hive, focusing on reducing the size of data written to disk while still allowing efficient row-wise processing. Parquet is a more mature format designed independently from Hadoop, with a strong emphasis on columnar storage and query optimization.

Data Layout: ORC supports both row-wise and columnar layouts within the same file, which allows it to provide flexibility in query performance tuning. Parquet stores data entirely in a columnar layout, making it highly efficient for analytical queries that scan columns rather than rows.

#### Performance Considerations

Write Performance: ORC writes are faster because they can write entire row groups at once. However, due to its flexibility, each row group might contain mixed types of data.
Read Performance: Parquet excels in read performance as it allows for efficient filtering and slicing based on column values without reading unrelated rows.

Use Cases:

ORC: Suitable for environments where you need a balance between write speed and flexibility, such as transactional databases or hybrid analytics workloads.
Parquet: Ideal for large-scale data warehousing and business intelligence scenarios where read performance is crucial, but the frequency of writes is lower.

In summary, while both ORC and Parquet aim to optimize data storage and processing in data lakes, their design priorities and use cases differ significantly. ORC offers more flexibility with mixed write-read workloads, whereas Parquet excels in optimized read scenarios typical of business intelligence queries.

#### Common mistakes
include assuming that one format is universally better than the other without considering specific use case requirements. Always evaluate your workload needs to choose the right storage format.

**Comparison:**

```
Format	Data Layout	Performance Considerations
ORC	Row-wise & Columnar	Flexible write-read, efficient row groups
Parquet	Fully Columnar	Optimized for read queries
```

**Code:**

```python
# Example Python code to write data using ORC and Parquet
import pyarrow as pa

# Writing to ORC
table = pa.Table.from_pydict({"column1": [1, 2, 3], "column2": ["a", "b", "c"]})
orc_writer = pa.orc.OrcFileWriter(pa.BufferOutputStream(), table.schema)
for batch in table.to_batches():
    orc_writer.write_batch(batch)
orc_writer.close()

# Writing to Parquet
parquet_writer = pa.parquet.ParquetWriter("example.parquet", table.schema, compression="snappy")
for batch in table.to_batches():
    parquet_writer.write_row_batch(batch)
parquet_writer.close()
```

**Example input / output:**

```text
Input:
- Write data using ORC and Parquet

Output:
- ORC file with row-wise and columnar layouts within the same file.
- Parquet file with columnar layout, optimized for read performance.
```

**Data:**

```json
{
  "table_schema": {
    "name": "example_table",
    "columns": [
      {"name": "column1", "type": "int"},
      {"name": "column2", "type": "string"}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Yes
8. No
9. Read Performance
10. Is the query column-oriented?
11. Fast filtering and slicing
12. Frequent row reads, lessefficient
13. Write Performance
14. Are writes frequent?
15. ORC: Fast write, mixed layouts
16. Parquet: Efficient read,columnar storage

