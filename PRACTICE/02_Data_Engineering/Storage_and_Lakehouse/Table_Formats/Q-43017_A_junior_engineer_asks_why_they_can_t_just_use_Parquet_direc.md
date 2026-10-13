---
type: "practice"
difficulty: "Intermediate"
---

# A junior engineer asks why they can't just use Parquet directly in S3 with a Glue crawler. Give the concrete failure scenarios a table format prevents.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Table Format Internals](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Table_Formats/Table_Format_Internals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the junior engineer's question, it’s essential to understand that while using Parquet files directly in Amazon S3 and relying on AWS Glue crawlers might seem straightforward, there are several limitations and potential issues that make other table formats like Iceberg, Delta Lake, or Hudi more suitable. Let's break down these failure scenarios to see why a structured table format is critical.

Firstly, let's define the key concepts:

Parquet: A columnar storage file format optimized for fast read performance.
Glue Crawler: An AWS service that automatically discovers and catalogs data stored in S3.
Iceberg, Delta Lake, and Hudi: Advanced table formats designed to handle complex data processing scenarios, including updates and deletes.

Now, let's explore the failure scenarios:

Concurrency Issues:

In a high-concurrency environment, multiple operations (e.g., writes, reads) can happen simultaneously. If Parquet files are used directly in S3 without a table format like Iceberg or Delta Lake, concurrent updates could lead to data corruption or inconsistencies.
For example, if two processes attempt to write to the same file at the same time, they might overwrite each other's changes, leading to incorrect data.

Incremental Writes:

Parquet files do not support atomic inserts; all writes must go through the entire file, which can be inefficient and error-prone in a streaming or real-time processing environment.
Iceberg allows for incremental appends, where new data is written to separate segments without overwriting existing ones. This feature ensures that reads continue to function correctly while new data is being appended.

Delete Operations:

Parquet files do not support efficient delete operations. To implement a delete, you would typically need to rewrite the entire file, which can be time-consuming and resource-intensive.
Iceberg supports soft deletes by marking records as deleted but keeping them in the table metadata. This allows for more efficient read operations while ensuring that deleted data is accounted for.

Schema Evolution:

When adding new columns or changing existing column types in Parquet files, it requires a complete rewrite of the file, which can be disruptive.
Iceberg and Delta Lake support schema evolution without requiring full rewrites. They maintain metadata about table schemas and changes over time, allowing for seamless updates.

Version Control:

Parquet files lack inherent versioning capabilities, making it difficult to track historical states of the data.
Iceberg provides a robust version control system that allows you to query different versions of the data at specific points in time, which is crucial for auditing and compliance.

#### Performance Considerations

While Parquet files excel in read performance due to their columnar structure, they can become less efficient when performing complex queries or operations.
Iceberg and Delta Lake are designed with both read and write efficiency in mind. They maintain a balance that ensures good performance across various operations.

In conclusion, while Parquet is excellent for read-heavy workloads where data integrity might not be the primary concern, it falls short in scenarios requiring concurrent writes, efficient deletes, schema evolution, and robust version control. The structured table formats like Iceberg, Delta Lake, or Hudi offer a more comprehensive solution by addressing these critical issues.

#### Common mistakes
include underestimating the complexity of real-world data processing requirements and assuming that simplicity (like using Parquet) will suffice in all scenarios.

**Comparison:**

```
Schema Evolution	Parquet	Iceberg
Atomic Inserts	No	Yes
Delete Operations	Inefficient	Soft Deletion (Marked as deleted)
Version Control	Limited or None	Robust and Trackable
```

**Code:**

```python
# Example code to illustrate schema evolution with Iceberg
from pyiceberg import Schema, Column

schema = Schema(
    columns=[
        Column(name="id", type=int),
        Column(name="name", type=str)
    ]
)

# Adding a new column
new_schema = Schema(
    columns=[
        Column(name="id", type=int),
        Column(name="name", type=str),
        Column(name="email", type=str)
    ]
)
```

**Example input / output:**

```text
Input:
Parquet file with schema: id (int), name (str)

Output:
Data corruption or inconsistency due to concurrent writes.

Desired Output:
Using Iceberg, schema evolution can be handled gracefully without rewriting the entire file.
```

**Data:**

```json
{
  "data": [
    {
      "id": 1,
      "name": "John Doe",
      "email": null
    },
    {
      "id": 2,
      "name": "Jane Smith",
      "email": "jane.smith@example.com"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Parquet File
6. Concurrent Writes?
7. Data Corruption
8. Read-Heavy Workload
9. Good Read Performance
10. Iceberg Table
11. Schema Evolution?
12. Graceful Handling
13. Complex Operations
14. Efficient Deletes, Soft Deletes
15. Read-Write Efficiency

