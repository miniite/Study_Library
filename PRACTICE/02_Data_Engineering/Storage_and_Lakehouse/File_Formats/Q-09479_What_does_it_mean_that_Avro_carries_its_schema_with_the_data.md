---
type: "practice"
difficulty: "Basic"
---

# What does it mean that Avro carries its schema with the data and why is that useful?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Row vs Columnar Formats](<../../../../02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Row_vs_Columnar_Formats.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this brief overview, we'll discuss what Avro means by carrying its schema with the data and why this feature is highly useful in a Data Engineer role. We'll start by understanding how Avro stores structured data efficiently, then delve into the implications of schema inclusion for data processing and management.

Avro is a columnar file format designed to store complex, nested data structures in an efficient manner. One of its key features is that it includes the schema directly within the data files themselves. This means that when you read Avro data, the necessary schema information is available alongside the actual data, making it easy for applications to understand and process the data without needing separate metadata storage.

To triage the question: the interviewer is probing your understanding of how different file formats store their metadata and the advantages this can bring. We will assume that you have a basic knowledge of columnar formats like Parquet and ORC, which do not carry schema information with them, contrasting Avro's approach.

**Code:**

```python
# Example Python code to read Avro data without needing separate metadata
from avro.datafile import DataFileReader
from avro.io import DatumReader

with open('example.avro', 'rb') as file:
    reader = DataFileReader(file, DatumReader())
    for record in reader:
        print(record)
```

**Example input / output:**

```

Given the above schema and a record, reading an Avro file would output:

Input (Avro data file):

AVRO
Copy
{
  "name": "Alice",
  "favorite_number": 7,
  "favorite_color": null
}

Output (parsed in Python):

JSON
Copy
{"name": "Alice", "favorite_number": 7, "favorite_color": null}
```

**Data:**

```
Avro Data Structure: A sample Avro data file might look like this:
JSON
Copy
{
  "namespace": "example.avro",
  "type": "record",
  "name": "User",
  "fields": [
    {"name": "name", "type": "string"},
    {"name": "favorite_number", "type": ["int", "null"]},
    {"name": "favorite_color", "type": ["string", "null"]}
  ]
}
Avro Record Example: A record might be:
JSON
Copy
{"name": "Alice", "favorite_number": 7, "favorite_color": null}
```

