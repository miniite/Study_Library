---
type: "practice"
difficulty: "Intermediate"
---

# How would you validate that a pipeline is not silently dropping rows somewhere between source and warehouse?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ End to End Exactly Once](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/End_to_End_Exactly_Once.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that no rows are being silently dropped in a big data processing pipeline, we need to focus on robust validation mechanisms. This involves both monitoring and logging practices at each stage of the pipeline to identify any potential data loss. At an intermediate level, we will walk through the steps required to set up these validations using Apache Kafka for ingestion and Apache Spark for processing.

The key idea is to implement a comprehensive tracking mechanism that can trace every row from its source to the warehouse, ensuring nothing gets dropped along the way. We'll start by understanding what data should be available at each stage, then implement proper checks.

**Comparison:**

```
Stage	Expected Rows
Kafka Source	1,000
Spark Processed	1,000
Warehouse Destination	1,000
```

**Code:**

```python
# Example validation code using Spark
from pyspark.sql import SparkSession

def validate_pipeline(spark):
    # Load raw data from Kafka
    kafka_df = spark.readStream.format("kafka") \
        .option("kafka.bootstrap.servers", "localhost:9092") \
        .option("subscribe", "raw-topic") \
        .load()

    # Transform and process the data (simple example)
    processed_df = kafka_df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")

    # Write to warehouse
    write_stream = processed_df.writeStream.format("parquet").outputMode("append").option("path", "/path/to/warehouse")

    # Start streaming query for validation
    write_query = write_stream.start()

    # Monitor the number of rows at each stage
    monitor_query = kafka_df.selectExpr("count(*) as row_count").writeStream.outputMode("complete") \
        .format("console").start()
```

**Example input / output:**

```text
Input:
- Kafka producer sends 1,000 rows

Output:
- Spark reads all 1,000 rows from Kafka
- Spark processes the data (transformations)
- All 1,000 rows are written to warehouse
```

**Data:**

```text
- Raw data in Kafka topic: `raw-topic`
- Processed and written to warehouse path: `/path/to/warehouse`
```

**Flow:**

1. Kafka Source
2. Spark Transformation
3. Warehouse Destination

