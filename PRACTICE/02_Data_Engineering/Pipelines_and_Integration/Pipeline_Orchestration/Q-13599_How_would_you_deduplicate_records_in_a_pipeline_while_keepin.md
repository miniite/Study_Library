---
type: "practice"
difficulty: "Deep"
---

# How would you deduplicate records in a pipeline while keeping only the most recent record per key?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Exactly Once Spark Writes](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Exactly_Once_Spark_Writes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To deduplicate records in a data pipeline and keep only the most recent record per key, we need to understand that this is a common problem in ETL (Extract, Transform, Load) processes where we want to ensure that our database or data warehouse contains the latest state of each entity. The solution involves using tools like Apache Airflow for workflow management and Apache Spark for efficient data processing.

In this case, we are tasked with maintaining a single recent record per key in our pipeline. This can be achieved by implementing a deduplication step using a combination of Airflow for managing the workflow and Spark for performing the actual deduplication logic. Let's break down the solution from first principles.

#### Triage

The problem requires us to implement a deduplication mechanism that ensures only the latest record is retained based on a specific key. This involves several steps: identifying unique keys, fetching the most recent data for each key, and filtering out older records. The core challenge here lies in efficiently handling large datasets and ensuring consistency across multiple runs of our pipeline.

#### Solution
Workflow Management with Airflow: We will use Apache Airflow to orchestrate the tasks involved in the deduplication process. Specifically, we can define a DAG (Directed Acyclic Graph) that handles data ingestion, transformation, and storage.
Data Ingestion: The first task is to ingest raw data from its source. This could be done using an API call or reading from a file system.
Deduplication Logic with Spark: Once the data is ingested, we will use Apache Spark to process it and retain only the most recent records based on their keys.
Step-by-Step Solution
Ingest Raw Data:
Define an Airflow task to ingest raw data from a source such as S3 or HDFS.
Transform with Spark:
Use a Spark job to process this ingested data and deduplicate it based on keys.
Implement a window function in Spark to find the most recent record for each key.

#### Example
Code
PYTHON
from pyspark.sql import SparkSession
from pyspark.sql.window import Window
import pyspark.sql.functions as F

# Initialize Spark session
spark = SparkSession.builder.appName("DeduplicationPipeline").getOrCreate()

# Sample raw data (assuming it is in a DataFrame)
raw_data = spark.read.csv("path/to/raw/data.csv")

# Define window specification to partition by key and order by timestamp
window_spec = Window.partitionBy("key").orderBy(F.col("timestamp").desc())

# Apply the deduplication logic using window functions
recent_records = raw_data.withColumn(
    "is_recent", F.row_number().over(window_spec) == 1
).filter(F.col("is_recent"))

# Drop unnecessary columns and save the result to a target location
recent_records.drop("is_recent").write.csv("path/to/output/deduplicated_data")

#### Closing Summary
+ Common Mistakes/Follow-Ups

To summarize, we have outlined how to implement deduplication in a data pipeline using Airflow for workflow management and Spark for efficient processing. The key steps involve ingesting the raw data, defining a window specification for sorting by timestamp, applying row_number() function to identify recent records, and filtering out older duplicates.

#### Common mistakes
include not properly handling the window functions, failing to account for partitioning by keys, or incorrectly saving intermediate states which can lead to data loss. In follow-up discussions, we could explore additional optimizations such as using micro-batch processing in Spark or leveraging existing libraries like Delta Lake for more robust deduplication mechanisms.

**Comparison:**

| Step | Action |
|------|--------|
| 1    | Ingest raw data from source. |
| 2    | Define window specification for deduplication. |
| 3    | Apply row_number() function to filter recent records. |
| 4    | Save the result without unnecessary columns. |

**Code:**

```python
from pyspark.sql import SparkSession
from pyspark.sql.window import Window
import pyspark.sql.functions as F

# Initialize Spark session
spark = SparkSession.builder.appName("DeduplicationPipeline").getOrCreate()

# Sample raw data (assuming it is in a DataFrame)
raw_data = spark.read.csv("path/to/raw/data.csv")

# Define window specification to partition by key and order by timestamp
window_spec = Window.partitionBy("key").orderBy(F.col("timestamp").desc())

# Apply the deduplication logic using window functions
recent_records = raw_data.withColumn(
    "is_recent", F.row_number().over(window_spec) == 1
).filter(F.col("is_recent"))

# Drop unnecessary columns and save the result to a target location
recent_records.drop("is_recent").write.csv("path/to/output/deduplicated_data")
```

**Example input / output:**

```text
Input:
+----+----------+-------------------+
| key | value    | timestamp         |
+----+----------+-------------------+
| A   | 10       | 2023-06-01T12:00Z |
| B   | 20       | 2023-05-30T14:00Z |
| C   | 30       | 2023-06-02T11:00Z |
| A   | 15       | 2023-06-01T13:00Z |
+----+----------+-------------------+

Output:
+----+----------+
| key | value    |
+----+----------+
| A   | 15       |
| B   | 20       |
| C   | 30       |
+----+----------+
```

**Data:**

```plaintext
Example raw data schema:
+----+----------+-------------------+
| key | value    | timestamp         |
+----+----------+-------------------+
| A   | 10       | 2023-06-01T12:00Z |
| B   | 20       | 2023-05-30T14:00Z |
| C   | 30       | 2023-06-02T11:00Z |
| A   | 15       | 2023-06-01T13:00Z |
+----+----------+-------------------+
```

**Flow:**

1. Yes
2. Ingest Raw Data
3. Use Airflow DAG?
4. Define Window Specification
5. Apply Spark DeduplicationLogic
6. Filter Recent Records
7. Saving Result
8. Complete

