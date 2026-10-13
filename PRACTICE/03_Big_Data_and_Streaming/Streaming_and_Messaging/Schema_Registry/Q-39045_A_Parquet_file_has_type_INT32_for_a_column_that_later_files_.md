---
type: "practice"
difficulty: "Intermediate"
---

# A Parquet file has type INT32 for a column that later files store as INT64. How does your query engine handle that on read and how do you prevent the mismatch at the pipeline level?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Protobuf vs Avro Evolution](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Protobuf_vs_Avro_Evolution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we face a common issue in data processing where schema evolution leads to type mismatches between different file versions. A Parquet file initially uses INT32 but later changes to INT64, potentially causing issues during read operations. The primary focus is on how the query engine handles this mismatch and strategies to prevent it at the pipeline level.

The query engine will handle the type mismatch by attempting to cast the data from INT32 to INT64 upon reading the newer Parquet files. However, if the value in the INT32 column exceeds the range of an INT64 (specifically, greater than 2^31 - 1), this could result in a loss or distortion of data.

To prevent such issues at the pipeline level, we need to ensure consistent schema validation and evolution practices. This involves versioning schemas, maintaining backward compatibility, and employing robust error handling mechanisms.

**Comparison:**

| Column | Old Schema (INT32) | New Schema (INT64) |
|--------|--------------------|-------------------|
| old_column | int                 | long              |
| other_column | string            | string             |

**Code:**

```python
# Example Python code snippet for reading Parquet files with type casting

from pyspark.sql import SparkSession

spark = SparkSession.builder.getOrCreate()

def read_parquet_with_type_cast(path):
    df = spark.read.parquet(path)

    # Assuming 'old_column' is the column with mismatched types
    df = df.withColumn('old_column', df['old_column'].cast('long'))

    return df

# Example usage
new_df = read_parquet_with_type_cast('/path/to/new/parquet/files')
```

**Example input / output:**

```

Example input and output:

Input:

Old Parquet file with schema {"fields": [{"name": "old_column", "type": "int"}, {"name": "other_column", "type": "string"}]}

Output (after type casting):

JSON
Copy
[
  {"old_column": 12345, "other_column": "value1"},
  {"old_column": -98765, "other_column": "value2"}
]
```

**Data:**

```

{ "schema": { "fields": [ {"name": "old_column", "type": "int"}, {"name": "other_column", "type": "string"} ] } }

CODE
Copy

```json
{
  "schema": {
    "fields": [
      {"name": "old_column", "type": "long"},
      {"name": "other_column", "type": "string"}
    ]
  }
}
```

**Flow:**

1. Schema Evolution
2. Yes
3. No
4. Read Parquet File
5. Is Column Type Mismatch?
6. Cast INT32 to INT64
7. Continue Processing
8. Version 1: INT32
9. Version 2: INT64

