---
type: "practice"
difficulty: "Basic"
---

# How does Parquet store its schema and why is it called self-describing?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Parquet Internals](<../../../../02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Parquet_Internals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Parquet stores its schema within the file itself, making it "self-describing." This means that anyone reading a Parquet file can understand what data types and column names are present without needing external metadata. Let's dive into how this works from first principles.

To start, we need to understand why self-describing is important. In traditional data storage formats like CSV or JSON, schema information might be stored separately or in the same files with a different format (like XML). Parquet takes a different approach by embedding this metadata directly into the file structure, which simplifies usage and reduces complexity.

Parquet stores its schema as metadata at the beginning of each file. This metadata includes details about the columns, their data types, and other relevant properties like compression settings. When reading a Parquet file, you can extract this metadata to know exactly what data you are dealing with, making it easier to work with large datasets.

Let's break down how this works step by step:

File Structure: The Parquet file consists of multiple sections: the metadata section, the actual data, and possibly footer information.
Metadata Section: This section contains a dictionary that describes the schema of the data stored in the file. It includes details such as column names, data types, number of rows, and statistics about the values.
Data Encoding: After the metadata, the file is divided into row groups. Each row group contains chunks of data for all columns. The encoding type (like dictionary or plain) can vary based on the data.

Here's a simple example to illustrate:

PARQUET
# Metadata Section
{
  "schema": {
    "columns": [
      {"name": "id", "type": "int64"},
      {"name": "value", "type": "double"}
    ]
  }
}

# Data Section
[
  [1, 3.14],
  [2, 2.71],
  [3, 0.5]
]

This example shows a Parquet file with two columns: id (an integer) and value (a double). The metadata section clearly defines these columns and their types.

Why is this important? Imagine you are working on a large ETL job where data flows through multiple systems. Knowing the schema directly from the file can save significant time in debugging and processing. Additionally, it reduces dependencies on external sources for metadata, making Parquet files more portable and easier to use across different environments.

In summary, Parquet's self-describing nature is achieved by embedding its schema within the file itself, ensuring that anyone who reads the file has all necessary information about its contents. This feature makes Parquet highly versatile and user-friendly in data processing pipelines.

**Comparison:**

```
Column Name	Type
id	int64
value	double
```

**Code:**

```python
# Example of reading metadata from a Parquet file using PyArrow
import pyarrow.parquet as pq

parquet_file = pq.ParquetFile('example.parquet')
schema = parquet_file.schema

print(schema)
```

**Example input / output:**

```

Input (metadata part of a Parquet file):

JSON
Copy
{
  "schema": {
    "columns": [
      {"name": "id", "type": "int64"},
      {"name": "value", "type": "double"}
    ]
  }
}

Output (parsed schema from metadata):

CODE
Copy
Schema: id -> int64, value -> double
```

**Data:**

```json
{
  "columns": [
    {"name": "id", "type": "int64"},
    {"name": "value", "type": "double"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Metadata Section
4. Is Metadata?
5. Read Schema
6. Proceed with Data Processing
7. Error: Invalid Parquet File

