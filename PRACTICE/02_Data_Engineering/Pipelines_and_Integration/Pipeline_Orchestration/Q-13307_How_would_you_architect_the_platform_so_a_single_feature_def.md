---
type: "practice"
difficulty: "Expert"
---

# How would you architect the platform so a single feature definition compiles to both a batch Spark job and a streaming job to guarantee identical semantics?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Shared Pipeline Platforms](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Orchestration_vs_Processing/Shared_Pipeline_Platforms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this architecture, we need to design a system where a single feature definition can be compiled into both a batch Spark job and a streaming Spark job while ensuring that they have identical semantics. This is a complex requirement but achievable by leveraging modern data engineering tools like Apache Airflow for orchestration and managing the different execution contexts of batch and stream processing.

#### Triaging the Question

The interviewer is probing our ability to design robust, flexible, and maintainable data pipelines that can handle both batch and streaming workloads. The core challenge here is ensuring consistency between these two types of jobs given their inherent differences in semantics and execution models.

#### Assumptions
Feature Definition: A feature definition includes all the logic required to compute a feature from raw input data, which could be structured or unstructured.
Batch vs. Streaming Execution: Batch processes typically involve reading historical data, processing it, and writing results back. Streaming jobs consume real-time data, process it as it arrives, and produce outputs immediately.
Semantics Preservation: The key challenge is maintaining identical semantics between batch and streaming operations.

#### Solution
1. Feature Definition Layer

Start by defining the feature logic in a common layer that can be shared across both batch and stream pipelines. This could be implemented as a function or a class:

PYTHON
def compute_feature(input_data):
    # Common computation logic for both batch and stream processing
    return processed_feature
2. Batch Spark Job

For the batch job, we will use Spark’s batch processing capabilities to read historical data, process it using the feature definition function, and write results back to storage.

PYTHON
# Example batch Spark job
def batch_job():
    # Read historical data
    input_df = spark.read.parquet("/path/to/historical_data")

    # Compute features
    output_df = input_df.withColumn("processed_feature", compute_feature(input_df))

    # Write processed data back to storage
    output_df.write.mode("overwrite").parquet("/path/to/output")
3. Streaming Spark Job

For the streaming job, we will adapt the same feature definition function and use Spark’s stream processing capabilities to consume real-time data.

PYTHON
# Example streaming Spark job
def streaming_job():
    # Create a streaming context
    spark_streaming = SparkSession.builder.appName("StreamingFeature").getOrCreate()

    # Read real-time data
    input_df = spark_streaming.readStream.format("kafka").option("kafka.bootstrap.servers", "localhost:9092").load().selectExpr("CAST(value AS STRING)")

    # Compute features
    output_df = input_df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)").withColumn("processed_feature", compute_feature(input_df))

    # Write processed data to Kafka topic for further processing or storage
    query = output_df.writeStream.outputMode("append").format("kafka").option("kafka.bootstrap.servers", "localhost:9092").option("topic", "output_topic").start()
4. Airflow Orchestration

Use Apache Airflow to orchestrate the batch and streaming jobs, ensuring that they are run in a coordinated manner.

AIRFLOW
from airflow import DAG
from airflow.operators.python_operator import PythonOperator

default_args = {
    'owner': 'data_engineer',
    'start_date': datetime(2023, 1, 1),
}

dag = DAG(
    'feature_processing',
    default_args=default_args,
    schedule_interval='@daily'
)

batch_task = PythonOperator(
    task_id='run_batch_job',
    python_callable=batch_job,
    dag=dag
)

streaming_task = PythonOperator(
    task_id='run_streaming_job',
    python_callable=streaming_job,
    dag=dag
)
5. Semantics Preservation

To ensure identical semantics, we need to consider the following:

State Management: Ensure that state is managed correctly in both batch and streaming contexts.
Error Handling: Implement robust error handling to catch and log any issues during processing.

#### Cons
istency Checks: Periodically validate results from both jobs against a known good dataset.
6. Deployment

Deploy the solution using containerization (Docker) and orchestration tools like Kubernetes, ensuring that resources are managed efficiently.

#### Closing Summary
+ Common Mistakes/Follow-Ups

This architecture ensures that a single feature definition can be compiled into both batch and streaming jobs with identical semantics. The key is to abstract the core logic in a reusable way and leverage Airflow for coordination. Some common mistakes include:

Not properly managing state across different processing contexts.
Failing to handle edge cases like data skew or late arrivals.
Overlooking consistency checks, which are crucial for validating that both jobs produce the same results.

Common follow-up questions might be about how we would scale this system, manage dependencies between tasks, and handle security and compliance concerns in a production environment.

**Comparison:**

| Input Data (Historical)       | Output Data (Processed Batch)    |
|------------------------------|---------------------------------|
| +----+----------------+        | +----+----------------+        |
| | id | raw_feature    |        | | id | processed_feature  |
| +====+================+        | +====+=================+
| | 1  | example1       |        | | 1   | processed1      |
| +----+----------------+        | +----+------------------+
| | 2  | example2       |        | | 2   | processed2      |
| +----+----------------+        | +----+------------------
| Input Data (Real-time Kafka)  | Output Data (Processed Stream) |
|------------------------------|--------------------------------|
| +----+-----------+            | +----+--------------+
| | key | value     |            | | id | processed_feature |
| +=====+===========+            | +====+==================+
| | 3   | example3  |            | | 3   | processed3      |
| +----+-----------+            | +----+------------------

**Code:**

```

def compute_feature(input_data): # Common computation logic for both batch and stream processing return processed_feature

def batch_job(): # Read historical data input_df = spark.read.parquet("/path/to/historical_data")

CODE
Copy
# Compute features
output_df = input_df.withColumn("processed_feature", compute_feature(input_df))

# Write processed data back to storage
output_df.write.mode("overwrite").parquet("/path/to/output")

def streaming_job(): # Create a streaming context spark_streaming = SparkSession.builder.appName("StreamingFeature").getOrCreate()

CODE
Copy
# Read real-time data
input_df = spark_streaming.readStream.format("kafka").option("kafka.bootstrap.servers", "localhost:9092").load().selectExpr("CAST(value AS STRING)")

# Compute features
output_df = input_df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)").withColumn("processed_feature", compute_feature(input_df))

# Write processed data to Kafka topic for further processing or storage
query = output_df.writeStream.outputMode("append").format("kafka").option("kafka.bootstrap.servers", "localhost:9092").option("topic", "output_topic").start()
CODE
Copy

```python
from airflow import DAG
from airflow.operators.python_operator import PythonOperator

default_args = {
    'owner': 'data_engineer',
    'start_date': datetime(2023, 1, 1),
}

dag = DAG(
    'feature_processing',
    default_args=default_args,
    schedule_interval='@daily'
)

batch_task = PythonOperator(
    task_id='run_batch_job',
    python_callable=batch_job,
    dag=dag
)

streaming_task = PythonOperator(
    task_id='run_streaming_job',
    python_callable=streaming_job,
    dag=dag
)
```

**Example input / output:**

```text
Batch Job:
Input Data (Historical):
+----+----------------+
| id | raw_feature    |
+====+================+
| 1  | example1       |
+----+----------------+
| 2  | example2       |
+----+----------------+

Output Data (Processed Batch):
+----+--------------+
| id | processed_feature |
+====+==================+
| 1  | processed1      |
+----+------------------+
| 2  | processed2      |
+----+------------------

Streaming Job:
Input Data (Real-time Kafka):
+----+-----------+
| key | value     |
+=====+===========+
| 3   | example3  |
+----+-----------+

Output Data (Processed Stream):
+----+--------------+
| id | processed_feature |
+====+==================+
| 3  | processed3      |
+----+------------------+
```

**Data:**

```json
{
  "historical_data": [
    {"id": 1, "raw_feature": "example1"},
    {"id": 2, "raw_feature": "example2"}
  ],
  "output_topic": {
    "type": "kafka",
    "name": "output_topic"
  }
}
```

**Flow:**

1. Feature Definition
2. Batch Spark Job
3. Streaming Spark Job
4. Batch Data Output
5. Streaming Data Output
6. Airflow Orchestration

