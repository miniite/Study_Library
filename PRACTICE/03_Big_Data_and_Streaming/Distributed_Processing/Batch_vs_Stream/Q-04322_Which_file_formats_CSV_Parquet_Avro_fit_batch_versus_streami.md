---
type: "practice"
difficulty: "Intermediate"
---

# Which file formats (CSV, Parquet, Avro) fit batch versus streaming and why?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Choosing Batch or Streaming](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Choosing_Batch_or_Streaming.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address which file formats—CSV, Parquet, or Avro—are suitable for batch processing versus streaming, we need to understand both the characteristics of these file formats and the requirements of each processing type.

Batch Processing vs. Streaming:

Batch Processing: This involves loading data in bulk, performing transformations and aggregations, and storing results in a final state.
Streaming Processing: This is about handling continuous streams of data in real-time or near-real-time, where data comes in chunks over time, requiring incremental processing.

File Formats Analysis:

CSV

Batch Processing:

#### Pros
Simple to read/write, widely supported by tools, easy for ad-hoc analysis.

#### Cons
Limited metadata support; not optimized for large datasets or complex queries. Each file is processed independently, leading to slower performance when dealing with large volumes of data.

Streaming Processing:

Not ideal due to its lack of metadata and sequential read/write nature, which does not support efficient incremental processing.
Parquet

Batch Processing:

#### Pros
Optimized for analytical queries; columnar storage allows fast scanning of subsets of columns. Efficient handling of large datasets with good performance.

#### Cons
Overhead in initial setup and compression/decompression processes.

Streaming Processing:

While Parquet can be used, it requires additional overhead due to its batch-oriented nature. However, with appropriate data slicing or windowing techniques, it can still be utilized for real-time processing.
Avro

Batch Processing:

#### Pros
Schema evolution capabilities; metadata embedded in files allowing better validation and understanding of the data structure.

#### Cons
Slightly more complex to implement compared to CSV, but generally easier than Parquet when dealing with schema changes over time.

Streaming Processing:

Avro is well-suited for streaming due to its dynamic schema support. It allows adding or modifying fields without breaking existing systems, making it ideal for evolving data pipelines.
Summary:

For batch processing, Parquet and Avro are more suitable compared to CSV due to their optimization for large-scale data storage and query efficiency. For streaming, Avro is particularly advantageous because of its schema evolution capabilities, whereas Parquet might still be used but with additional overhead considerations.

#### Common Mistakes
Overlooking the importance of metadata in real-time systems.
Ignoring the performance implications when handling large datasets in CSV or Parquet without proper optimizations.
Not considering Avro's flexibility for evolving schemas in dynamic environments.

**Comparison:**

```
File Format	Batch Processing Pros	Batch Processing Cons	Streaming Pros	Streaming Cons
CSV	Simple	Limited metadata	None	Not efficient
Parquet	Optimized for queries	Overhead	Can be used	Additional overhead
Avro	Schema evolution	Complex implementation	Flexible	More complex
```

**Code:**

```python
# Example code snippet to read/write Parquet and Avro files

import pandas as pd
from pyarrow import parquet as pq
from avro.datafile import DataFileReader, DataFileWriter
from avro.io import DatumReader, DatumWriter
from io import BytesIO

# Reading a Parquet file
parquet_file = pq.ParquetFile("data.parquet")
df = pd.read_parquet("data.parquet")

# Writing to Parquet
data = {"key": [1, 2], "value": ["a", "b"]}
table = pa.Table.from_pandas(pd.DataFrame(data))
pq.write_table(table, "output.parquet")

# Reading a Avro file
reader = DataFileReader(open("data.avro", "rb"), DatumReader())
for record in reader:
    print(record)
```

**Example input / output:**

```text
Input (CSV):
id,name
1,Alice
2,Bob

Output (Parquet):
Read Parquet file and perform query:
SELECT name FROM data WHERE id = 1
Result: Alice

Input (Avro):
Reading Avro record with schema evolution:
New field added: "email" -> {"id": 3, "name": "Charlie", "email": "charlie@example.com"}
```

**Data:**

```json
{
  "schema": {
    "type": "record",
    "name": "ExampleRecord",
    "fields": [
      {"name": "id", "type": "int"},
      {"name": "name", "type": "string"}
    ]
  },
  "data": [{"id": 1, "name": "Alice"}, {"id": 2, "name": "Bob"}]
}
```

**Flow:**

1. Yes
2. No
3. Read CSV
4. Is Metadata Important?
5. Use Avro
6. Use Parquet
7. Handle Schema Evolution
8. Handle Large VolumesEfficiently

