---
type: "practice"
difficulty: "Deep"
---

# A team reports S3 request costs spiking after adopting Iceberg. Which access patterns (LIST, manifest reads, small files) drive request charges and how do you cut them?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Table Format Internals](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Table_Formats/Table_Format_Internals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the spike in S3 request costs experienced by a team using Iceberg, we need to understand which access patterns are driving these costs and explore strategies to mitigate them. Iceberg uses three main access methods: List operations, manifest reads, and small file handling. By optimizing each of these, we can significantly reduce request charges.

Iceberg works with S3 storage in a way that allows for efficient data management and querying without the overhead associated with traditional table formats like Parquet or ORC. However, improper usage can lead to increased costs due to frequent S3 operations. The key is to ensure that our access patterns are optimized for cost efficiency while maintaining query performance.

Triaging the Problem

The core issue here is understanding why Iceberg might be generating more S3 requests than before its adoption. Typically, Iceberg's manifest reads and list operations are responsible for most of these requests. Additionally, small files created during incremental writes can also contribute to higher costs. We need to analyze the access patterns and identify areas where optimization is needed.

#### First-Principles Solution
List Operations: These occur when a query needs to determine which files belong to a table segment. Frequent list operations can be expensive, especially if many segments are being queried.
Manifest Reads: Manifest files in Iceberg act as indexes that map logical file names to physical S3 paths. Reading these manifests is necessary for queries but can become costly if the number of manifest reads increases.
Small Files Handling: During incremental writes or compaction, smaller files can be created, leading to a higher number of S3 requests.

To mitigate these costs, we need to:

Optimize List Operations: Ensure that our queries are as specific and efficient as possible. Avoid using wildcard characters in queries.
Reduce Manifest Reads: Optimize the way data is written and read by Iceberg. For example, using more compact manifest files or reducing the number of segments per table.
Handle Small Files Efficiently: Implement compaction strategies to merge small files into larger ones, reducing the overall file count.

#### Closing Summary

By understanding and optimizing these access patterns, we can significantly reduce S3 request charges while maintaining query performance. The key is to balance cost efficiency with data accessibility and query speed.

#### Common Mistakes/Follow-Ups
Overly Broad Queries: Using wildcard characters in queries can lead to unnecessary list operations.
Inefficient Compaction Strategies: Failing to implement compaction strategies can result in a large number of small files.
Ignoring Segment Management: Improper management of table segments can lead to frequent manifest reads.

Understanding the underlying mechanisms and optimizing access patterns is crucial for managing costs effectively while leveraging Iceberg's benefits.

**Comparison:**

| File Size (MB) | Count |
|---------------|-------|
| <5             | 80    |
| >100           | 2     |

**Code:**

```python
# Example code snippet for understanding list operations and manifest reads
from iceberg import Table, FileScanTask

def optimize_queries(table: Table):
    # Avoid wildcard characters in queries
    result = table.scan(column="id", value="1*")

    # Optimize compaction to reduce small files
    table.compact()
```

**Example input / output:**

```text
Request before optimization: 500 list operations, 300 manifest reads
Optimized request: 200 list operations, 150 manifest reads

Input:
- Original query: table.scan(column="id", value="*")
Output:
- Optimized query: table.scan(column="id", value="1")

Request before compaction: 100 small files
Optimized request: 20 merged large files
```

**Data:**

```json
{
  "segment": {
    "files": ["file1.parquet", "file2.parquet"],
    "manifests": ["manifest-001.json"]
  },
  "query": {
    "type": "list",
    "segments": [1, 2, 3],
    "files": ["file1.parquet", "file2.parquet", "file3.parquet"]
  }
}
```

**Flow:**

1. Query Execution
2. List Operation
3. Manifest Read
4. Small File Handling
5. Compaction Strategy

