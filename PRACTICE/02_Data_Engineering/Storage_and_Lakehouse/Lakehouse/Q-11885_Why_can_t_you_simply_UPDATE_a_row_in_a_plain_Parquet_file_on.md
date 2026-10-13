---
type: "practice"
difficulty: "Intermediate"
---

# Why can't you simply UPDATE a row in a plain Parquet file on a lake?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Columnar Formats in the Lake](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Columnar_Formats_in_the_Lake.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why you can’t simply update a row in a plain Parquet file, we need to delve into how Parquet files are structured and designed. In essence, Parquet is an efficient columnar storage format optimized for read-heavy workloads, not for frequent updates. The core reason lies in the immutable nature of Parquet and its design principles.

Parquet files are built on the principle that once data is written to a file, it should remain unchanged. This immutability ensures consistency and allows for efficient processing using MapReduce or other batch processing frameworks. If you were to allow direct updates, this would lead to several issues:

Data Integrity: Direct row-level updates can introduce inconsistencies between different files that might reference the same data.
Performance Overhead: Updating individual rows in a large file can be very inefficient and may require creating an entirely new file, defeating the purpose of Parquet's efficient storage.

Let’s break this down from first principles:

#### Triage

The question is fundamentally asking about the limitations of Parquet files when it comes to updating data. The interviewer is probing your understanding of the immutable nature of Parquet and how this affects operations like updates.

#### Assumptions
and Reasoning
Immutable Data: Parquet is designed with immutability in mind, meaning once a file is written, its contents cannot be changed.
Row-Level Operations: Unlike traditional database systems that support transactional updates, Parquet does not natively support row-level operations like UPDATE.
Efficiency and Performance: Parquet optimizes for read-heavy workloads by storing data in columns, which allows for fast scanning. This design choice makes it impractical to support direct row-level updates.
Step-by-Step Solution

Understanding the File Structure:

A Parquet file is a collection of rows grouped into blocks.
Each column within these blocks is stored contiguously, making scans faster and more efficient.

Immutable Data Handling:

If you try to update a row directly, it would require rewriting that entire row or even the block containing it.
This process can be slow and resource-intensive, especially for large files.

Alternative Solutions:

Appending New Data: Instead of updating, you could append new data to the existing file or create a new file entirely.
Delta Lake or Apache Iceberg: These systems build on top of Parquet but provide mechanisms like transaction logs and compaction to manage updates efficiently.

#### Closing Summary

In summary, you can’t update rows in plain Parquet files due to their immutable design. This ensures data integrity and efficiency for read-heavy workloads. However, there are alternative solutions that leverage Parquet’s strengths while providing the flexibility needed for updates.

#### Common Mistakes/Follow-Ups
Assuming In-Memory Updates: Be wary of thinking you can perform in-memory updates on a Parquet file without writing new data.
Ignoring Alternative Solutions: Don’t overlook frameworks like Delta Lake or Iceberg, which are designed to handle such scenarios while maintaining the benefits of Parquet.

**Comparison:**

```
Column	Old Data (Parquet)	New Data
column1	[1, 2]	[3, 4]
column2	["a", "b"]	["c", "d"]
```

**Code:**

```python
# Example code for appending new data to an existing Parquet file using pandas and PyArrow
import pyarrow.parquet as pq
import pandas as pd

# Load existing data
existing_data = pq.read_table('path/to/parquet/file').to_pandas()

# New data to append
new_data = pd.DataFrame({'column1': [1, 2], 'column2': ['a', 'b']})

# Append new data to the existing DataFrame
updated_data = existing_data.append(new_data)

# Write updated data back to Parquet file
updated_table = pa.Table.from_pandas(updated_data)
pq.write_table(updated_table, 'path/to/updated/file.parquet')
```

**Example input / output:**

```text
Input:
- Existing Parquet file with data.
- New data to append.

Output:
- Updated Parquet file containing both the original and new data.
```

**Data:**

```json
{
  "column1": [1, 2],
  "column2": ["a", "b"]
}
```

**Flow:**

1. No
2. Yes
3. Read Existing Parquet File
4. Is Data Mutable?
5. Append New Data
6. Write Updated Data to NewParquet File
7. Error: Parquet is Immutable forUpdates

