---
type: "practice"
difficulty: "Intermediate"
---

# How would you migrate a legacy Hive-on-MapReduce ETL to PySpark and what compatibility issues (SerDe partition metadata) do you expect?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Structured Streaming in PySpark](<../../../../TOOLS/Apache_Spark/PySpark/Structured_Streaming_in_PySpark.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To migrate a legacy Hive-on-MapReduce ETL pipeline to PySpark, we need to understand that this is a significant shift in processing framework and architecture. The primary goal is to ensure data integrity and performance while leveraging modern Spark features and improved scalability. Here’s a step-by-step approach to tackle the migration, along with potential compatibility issues:

#### Overview
We will start by assessing the current Hive-on-MapReduce ETL pipeline, identifying critical components like data sources, transformations, and sinks. Then we’ll refactor these steps using PySpark while ensuring that metadata (such as SerDe and partition information) is correctly preserved.

#### Triage
The key issues to address are:

Data compatibility: Ensuring the same data transformations can be applied in PySpark.
Metadata handling: Preserving partitioning, SerDe configurations, and other schema details necessary for Hive tables.
Performance optimization: Leveraging PySpark’s parallelism and optimizations.

Step-by-Step Solution:

Assessment: Evaluate the current ETL pipeline to understand data sources (HDFS, HBase, etc.), transformations, and targets.
Setup Environment: Ensure you have a robust PySpark environment configured with necessary dependencies like Spark SQL, Hive integration, and UDF support.
Data Loading:
PYTHON
# Example of loading data from an HDFS directory using PySpark
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("HiveToPySparkMigration").getOrCreate()
df = spark.read.format('orc').load('/path/to/dataset')
Transformation: Refactor the ETL steps. For instance, if you were using UDFs in Hive:
PYTHON
# Example of a PySpark UDF
from pyspark.sql.functions import udf
from pyspark.sql.types import IntegerType

@udf(returnType=IntegerType())
def transform_column(column_value):
    return int(column_value) * 2

transformed_df = df.withColumn("transformed_col", transform_column(df.column_name))
Saving Data: Save the processed data back to HDFS in a format compatible with Hive:
PYTHON
# Example of writing data back to HDFS as ORC file, which is also compatible with Hive
transformed_df.write.format('orc').mode('overwrite').save('/path/to/output')

Handling Metadata:

Partitioning: Ensure that partition directories and files are correctly managed in PySpark.
SerDe Compatibility: Some SerDe configurations might not be directly compatible. For example, if you were using ORC format with specific Hive SerDe settings:
PYTHON
# Example of reading an ORC file with custom SerDe settings
df = spark.read.orc('/path/to/dataset', schema='col1: STRING, col2: INT')

#### Closing Summary
By carefully refactoring the ETL pipeline in PySpark and ensuring that metadata and data formats are correctly handled, we can migrate a legacy Hive-on-MapReduce ETL to PySpark while maintaining performance and data integrity.

#### Common Mistakes/Follow-Ups

Ignoring schema changes: Ensure all schema details (including partitioning) are accurately transferred.
Overlooking performance optimizations in PySpark that could improve the migration.
Not testing thoroughly both on local and production environments to catch any unforeseen issues.

**Comparison:**

```
Input DF Schema	Output DF Schema
col1: STRING	col2: INT
col2: INT	transformed_col
```

**Code:**

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("HiveToPySparkMigration").getOrCreate()
df = spark.read.format('orc').load('/path/to/dataset')

@udf(returnType=IntegerType())
def transform_column(column_value):
    return int(column_value) * 2

transformed_df = df.withColumn("transformed_col", transform_column(df.column_name))

# Example of saving data back to HDFS as ORC file
transformed_df.write.format('orc').mode('overwrite').save('/path/to/output')
```

**Example input / output:**

```text
- **Input**:
  - `df = spark.read.format('orc').load('/path/to/dataset')`
  - `transformed_df = df.withColumn("transformed_col", transform_column(df.column_name))`

- **Output**:
  - `transformed_df.write.format('orc').mode('overwrite').save('/path/to/output')`
```

**Data:**

```json
{
  "dataset": {
    "file_path": "/path/to/dataset",
    "format": "orc"
  },
  "output_path": "/path/to/output"
}
```

**Flow:**

1. Yes
2. No
3. Read Data
4. Is SerDe Custom?
5. Configure SerDe
6. Transform and Save
7. Direct Load/Save

