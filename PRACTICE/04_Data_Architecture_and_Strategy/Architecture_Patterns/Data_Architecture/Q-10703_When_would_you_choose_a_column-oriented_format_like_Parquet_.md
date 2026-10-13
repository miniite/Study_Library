---
type: "practice"
difficulty: "Intermediate"
---

# When would you choose a column-oriented format like Parquet over row-oriented Avro for your training data, and when the reverse for streaming ingestion?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Vector Data in the Lakehouse](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Vector_Data_in_the_Lakehouse.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between using a column-oriented format like Parquet versus a row-oriented format like Avro for training data or streaming ingestion, several factors come into play, including query performance, storage efficiency, and data processing workflows. For ML Engineers with intermediate experience, understanding these trade-offs is crucial.

#### Triaging the question
The core of this inquiry revolves around choosing between Parquet (column-oriented) and Avro (row-oriented) based on specific use cases. Parquet excels in scenarios where columnar access patterns are frequent, such as analytics and training data processing. In contrast, Avro supports more flexible schema evolution and is better suited for streaming ingestion where data arrives in a row format.

#### Assumptions
We assume you are familiar with the characteristics of both formats:

Parquet organizes data by columns, which can optimize read performance.
Avro stores data in rows but allows dynamic schema changes over time.

Step-by-step reasoning from first principles:

Training Data Processing (e.g., ML Model Training)

Why use Parquet?
High query performance: Columnar storage optimizes access to specific columns.
Efficient compression: Reduces storage footprint, which is crucial for large datasets.
Performance on batch processing: Excellent for read-heavy workloads like ETL and training pipelines.

Streaming Ingestion (e.g., Real-Time Data Feed)

Why use Avro?
Flexible schema handling: Supports dynamic addition or removal of fields, which is valuable in evolving datasets.
Schema versioning: Facilitates backward compatibility without reprocessing old data.
Incremental updates: Efficient for real-time updates and partial writes.

#### Closing summary
In summary, the choice between Parquet and Avro hinges on the specific use case. For training data processing where performance and storage efficiency are paramount, Parquet is often preferred. Conversely, for streaming ingestion that requires flexible schema handling and incremental updates, Avro is a better fit.

#### Common mistakes/follow-ups

Mistake: Choosing based solely on perceived speed without considering the actual workload. Always profile real-world scenarios.

#### Follow-up
Discussing other factors like data governance policies, ease of integration with existing tools (like Apache Spark), and cost implications.

**Comparison:**

```
Schema	Parquet	Avro
Columnar Storage	Yes	No
High Query Performance	Yes	No
Efficient Compression	Yes	No
Flexible Schema Handling	No	Yes
Backward Compatibility	Limited	Good
```

**Code:**

```
Example Parquet write operation in Python using PyArrow

import pyarrow.parquet as pq

data = { "feature1": [1, 2, 3], "feature2": ["A", "B", "C"] }

table = pa.Table.from_pydict(data) pq.write_table(table, "data.parquet")

CODE
Copy

```python
# Example Avro write operation in Python using pyavro
from avro.datafile import DataFileWriter
from avro.io import DatumWriter
import json

schema_str = """
{
  "type": "record",
  "name": "ExampleRecord",
  "fields": [
    {"name": "feature1", "type": ["null", "int"]},
    {"name": "feature2", "type": ["null", "string"]}
  ]
}
"""

writer = DataFileWriter(open("data.avro", "wb"), DatumWriter(), json.loads(schema_str))
record = {"feature1": 1, "feature2": "A"}
writer.append(record)
writer.close()
```

**Example input / output:**

```

Input (Parquet)

PLAINTEXT
Copy
Feature1 | Feature2
1        | A
2        | B
3        | C

Output (Avro)

PLAINTEXT
Copy
{"feature1": 1, "feature2": "A"}
{"feature1": 2, "feature2": "B"}
{"feature1": 3, "feature2": "C"}
```

**Data:**

```json
{
  "example_parquet_schema": {
    "name": "ExampleRecord",
    "type": "record",
    "fields": [
      {"name": "feature1", "type": ["null", "int"]},
      {"name": "feature2", "type": ["null", "string"]}
    ]
  },
  "example_avro_schema": {
    "type": "record",
    "name": "ExampleRecord",
    "fields": [
      {"name": "feature1", "type": ["null", "int"]},
      {"name": "feature2", "type": ["null", "string"]}
    ]
  }
}
```

**Flow:**

1. Column-oriented Data
2. Parquet
3. Row-oriented Data
4. Avro

