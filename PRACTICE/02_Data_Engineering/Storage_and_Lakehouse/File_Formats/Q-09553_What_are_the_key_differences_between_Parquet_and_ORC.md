---
type: "practice"
difficulty: "Intermediate"
---

# What are the key differences between Parquet and ORC?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Parquet Internals](<../../../../02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Parquet_Internals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In this response, we will delve into the key differences between Parquet and ORC, two widely used columnar storage formats in big data ecosystems. Both formats are designed to optimize query performance by storing data in a columnar manner, but they differ significantly in terms of storage efficiency, scalability, metadata handling, and use cases.

#### Triage

The interviewer is probing your understanding of the fundamental differences between Parquet and ORC, which are essential for efficient data processing. You will need to explain the key features that distinguish these formats while highlighting their strengths and limitations.

#### First-Principles Solution
Storage Efficiency

Both Parquet and ORC use columnar storage, but they differ in how they compress and store data. Parquet uses dictionary encoding for repeated values within a column, allowing it to reduce the number of bytes needed to represent these values. ORC, on the other hand, combines dictionary encoding with run-length encoding (RLE) for sequences of identical values. This results in ORC often achieving better compression ratios.

Metadata and Flexibility

Parquet has a richer metadata format that allows it to store additional information such as statistics about each column, which can be used for query optimization. ORC is more flexible and supports a broader range of data types and encodings, making it suitable for scenarios where the schema might change frequently.

Performance

Both formats are optimized for fast query performance through their use of indexing and filtering capabilities. However, Parquet excels in handling complex queries due to its richer metadata and support for predicate pushdown (pushing filter conditions down to storage level). ORC, while less flexible in schema changes, can be more efficient for smaller datasets and simpler queries.

Evolution and Community Support

Parquet has been around longer and is part of the Apache Hadoop ecosystem. It benefits from a large community and extensive documentation, making it easier to integrate into existing big data workflows. ORC, on the other hand, was developed by Cloudera and later became an Apache project. It offers more flexibility in terms of schema evolution but might not have as broad community support compared to Parquet.

#### Closing Summary

In summary, while both Parquet and ORC are excellent columnar storage formats, they cater to different needs. Parquet is ideal for complex queries and scenarios where metadata-rich schemas are necessary, whereas ORC provides a more flexible schema that can handle changes in data structures more gracefully.

#### Common Mistakes/Follow-Ups
Mistake: Assuming one format is universally superior without considering the specific use case.

#### Follow-Up
Discussing scenarios where one might be preferred over the other, such as frequent schema changes or complex query requirements.

**Comparison:**

| Key Differences | Parquet                    | ORC                      |
|-----------------|----------------------------|--------------------------|
| **Metadata**    | Richer metadata            | Less rigid schema       |
| **Compression** | Dictionary encoding        | Runs-length + dictionary |
| **Flexibility** | Schema changes difficult   | Schema changes easier    |
| **Performance** | Excellent for complex Qs  | Good but simpler queries |

**Code:**

```python
# Example Parquet and ORC Schema Definitions (Simplified)
parquet_schema = {
    "name": "Parquet",
    "columns": [
        {"name": "id", "type": "int"},
        {"name": "value", "type": "double"}
    ],
    "metadata": {
        "statistics": {
            "id": {"min": 1, "max": 1000}
        }
    }
}

orc_schema = {
    "name": "ORC",
    "columns": [
        {"name": "id", "type": "int"},
        {"name": "value", "type": "double"}
    ],
    "metadata": {}
}
```

**Example input / output:**

```text
# Example Input-Output for Parquet and ORC Data Read (Simplified)
parquet_read = [
    [1, 3.14],
    [2, 6.28],
    [3, 9.42]
]

orc_read = [
    [1, 3.14],
    [2, 6.28],
    [3, 9.42]
]
```

**Data:**

```json
{
  "parquet_data": [
    {"id": 1, "value": 3.14},
    {"id": 2, "value": 6.28},
    {"id": 3, "value": 9.42}
  ],
  "orc_data": [
    {"id": 1, "value": 3.14},
    {"id": 2, "value": 6.28},
    {"id": 3, "value": 9.42}
  ]
}
```

**Flow:**

1. Parquet
2. Richer Metadata
3. Dictionary Encoding
4. Complex Query Support
5. ORC
6. Flexible Schema
7. Run-Length + DictionaryEncoding
8. Frequent Schema Changes

