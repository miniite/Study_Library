---
type: "practice"
difficulty: "Intermediate"
---

# Explain when you'd choose Parquet versus Avro for a given stage of your pipeline. Where does row-oriented Avro win and where does columnar Parquet win?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Vector Data in the Lakehouse](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Vector_Data_in_the_Lakehouse.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a data engineering context, choosing between Parquet and Avro depends on the specific requirements and characteristics of each stage in the pipeline. Parquet is a columnar storage format ideal for large-scale batch processing, whereas Avro excels with its schema evolution capabilities, making it more flexible for structured data that may change over time.

Avro wins when you need to support evolving schemas, ensuring compatibility across different systems or versions without breaking existing code. Its compact and self-describing nature also aids in efficient serialization and deserialization processes. On the other hand, Parquet shines due to its columnar storage, which optimizes read performance by allowing parallel processing of data columns.

Let's dive deeper into the first principles behind these choices.

**Comparison:**

```
Format	Schema Evolution Support	Columnar Storage	Read Performance
Avro	High (Incremental)	Limited	Good for specific cases
Parquet	Low (Initial Design Only)	Yes	Excellent in large-scale batch processing
```

**Code:**

```python
# Example code for reading Avro and Parquet files (not executable)
import pyarrow.parquet as pq
from avro.datafile import DataFileReader, DataFileWriter
from avro.io import DatumReader, DatumWriter

def read_avro_file(file_path):
    with open(file_path, 'rb') as f:
        reader = DataFileReader(f, DatumReader())
        for record in reader:
            print(record)
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Avro Schema Evolution
6. Does the data schema changeover time?
7. Use Avro for evolving schemas
8. Check if columnar storage isneeded
9. Choose Parquet for high readperformance
10. Consider other factors like sizeand processing needs

