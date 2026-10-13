---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd handle schema evolution safely in a bronze layer that ingests from a source that adds/renames columns without warning.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Delta Live Tables and Serving](<../../../../TOOLS/Databricks/Databricks_Platform/Delta_Live_Tables_and_Serving.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle schema evolution safely in a Databricks bronze layer, we need to ensure that our data pipeline can adapt to changes such as new or renamed columns in real-time sources. At this senior level, we'll focus on building a robust and scalable solution that minimizes downtime and ensures data integrity.

First, let’s define the problem: The source dataset is likely a stream of events from an external system that may add or rename columns without prior notice. This necessitates a dynamic schema management strategy to handle these changes seamlessly in our bronze layer.

#### Triaging the Question

The core question revolves around robust schema evolution strategies for ingesting streaming data into Databricks while maintaining data integrity and minimizing disruptions. Key aspects include:

Handling new columns: Ensuring they are included without overwriting existing data.
Renaming columns: Mapping old column names to new ones, preserving historical data.
Error handling: Detecting and logging schema mismatches before committing changes.

#### Assumptions
The source system can send metadata about column additions or renamings.
We have a mechanism for temporarily storing incoming events during schema evolution.
Our bronze layer uses Delta Lake for storage, leveraging its capabilities to manage schemas dynamically.
Step-by-Step Solution
1. Schema Definition and Versioning

Define initial schema in Databricks using PySpark or Spark SQL:

PYTHON
from pyspark.sql.types import StructType, StructField, StringType, LongType

initial_schema = StructType([
    StructField("id", LongType(), True),
    StructField("name", StringType(), True)
])
2. Stream Processing with Delta Lake

Use a streaming approach to ingest data into Delta Lake:

PYTHON
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("BronzeLayer").getOrCreate()
streaming_df = spark.readStream.schema(initial_schema).option("maxFilesPerTrigger", 1).csv("/path/to/source")
3. Handling New Columns

When new columns are introduced, update the schema and repartition the DataFrame:

PYTHON
def handle_new_columns(schema):
    updated_schema = add_new_columns_to_schema(initial_schema, schema)
    return updated_schema

def add_new_columns_to_schema(existing_schema, new_schema):
    # Logic to merge existing and new schemas
    pass

final_schema = handle_new_columns(new_schema)
streaming_df.writeStream.format("delta").option("checkpointLocation", "/path/to/checkpoint").start("/path/to/delta_table")
4. Column Renaming

For column renamings, map old columns to new ones during ingestion:

PYTHON
def handle_column_renames(df):
    return df.withColumnRenamed("old_name", "new_name")

streaming_df = handle_column_renames(streaming_df)
5. Error Handling and Logging

Implement error handling to detect schema mismatches:

PYTHON
from pyspark.sql.functions import col, when

def log_schema_mismatch(df):
    mismatched_cols = [col(c) for c in df.columns if not c.startswith("valid_prefix_")]
    df.withColumn("error", when(col("*").isin(mismatched_cols), "Schema Mismatch").otherwise(None)).writeStream.format("console").start()

log_schema_mismatch(streaming_df)

#### Closing Summary

By dynamically managing schemas and using streaming capabilities of Delta Lake, we can handle schema evolution safely in a Databricks bronze layer. This approach ensures that our pipeline adapts to changes without downtime or data loss, maintaining high integrity and reliability.

#### Common Mistakes/Follow-Ups
Overwriting Data: Ensure new columns are added instead of overwriting existing ones.
Schema Mismatch Logging: Always log schema mismatches for debugging and alerting.

#### Performance Considerations
Monitor performance impact when handling large numbers of column changes.

**Code:**

```python
from pyspark.sql.types import StructType, StructField, StringType, LongType

initial_schema = StructType([
    StructField("id", LongType(), True),
    StructField("name", StringType(), True)
])

def handle_new_columns(schema):
    updated_schema = add_new_columns_to_schema(initial_schema, schema)
    return updated_schema

def add_new_columns_to_schema(existing_schema, new_schema):
    # Logic to merge existing and new schemas
    pass

final_schema = handle_new_columns(new_schema)

from pyspark.sql.functions import col, when

def log_schema_mismatch(df):
    mismatched_cols = [col(c) for c in df.columns if not c.startswith("valid_prefix_")]
    df.withColumn("error", when(col("*").isin(mismatched_cols), "Schema Mismatch").otherwise(None)).writeStream.format("console").start()
```

