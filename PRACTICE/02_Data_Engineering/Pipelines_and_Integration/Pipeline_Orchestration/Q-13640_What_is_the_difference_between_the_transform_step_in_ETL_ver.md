---
type: "practice"
difficulty: "Basic"
---

# What is the difference between the transform step in ETL versus ELT and where does the transform happen in each?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ File and Table Formats for Features](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/File_and_Table_Formats_for_Features.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Data Engineering, ETL and ELT are two different paradigms used to process data before it's stored or analyzed. The key difference lies in when the transformation step happens during the data pipeline processing.

TL;DR:
ETL (Extract, Transform, Load) involves extracting raw data from sources, transforming it into a format suitable for analysis, and then loading it into a data warehouse.
ELT (Extract, Load, Transform) extracts raw data directly into a data warehouse or a cloud storage platform like S3, then applies transformations in batch or stream processing.

#### Triage
The Question

The question is asking about the differences between ETL and ELT paradigms. Specifically, it probes your understanding of when transformations occur within these processes—whether before (ETL) or after (ELT)—and where exactly they happen.

#### First-Principles Solution

Extract: Both ETL and ELT start by extracting data from various sources.

Transform:

In ETL, the transformation step happens immediately after extraction but before loading into a database or data warehouse.
In ELT, the transformation step occurs post-extraction, often during loading or processing within the target system.

Load:

ETL loads transformed data directly into a database or data warehouse.
ELT stores raw data first and then applies transformations in batch or real-time processing environments.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

The main difference between ETL and ELT is the timing of the transformation step. ETL processes are more suited to scenarios where complex transformations are necessary before storing the data, as they require specialized tools like Apache Spark for handling large volumes of transformed data. ELT, on the other hand, can be beneficial in environments that prefer a simpler setup, as it allows raw data to be stored first and then processed using tools like SQL or custom scripts.

#### Common mistakes
include misunderstanding when transformations should occur or not realizing that ETL systems often require more upfront configuration compared to ELT setups. Follow-up questions might explore specific use cases where one paradigm would be preferred over the other based on factors such as data volume, complexity of transformations, and available processing resources.

#### Example
Context

#### Cons
ider a scenario involving customer transaction logs from multiple retailers:

ETL: Extracts raw logs, transforms them (e.g., aggregating sales by date), and loads the aggregated data into a data warehouse.
ELT: Exports raw logs to S3, then uses a batch processing job with Apache Spark to aggregate the data before loading it back into the data warehouse.
Self-Check:

This solution is comprehensive, addressing both ETL and ELT processes accurately and providing context for their respective use cases.

**Comparison:**

```
Extract
ETL Transform
ELT Load
Load ETL Data Warehouse
Process & Load ELT
Extract
ETL Load
Raw Data Sources
Fine-grained Control overTransformations
Transform
ETL
Transform
ELT
```

**Code:**

```text
# Example code snippet in Python using Airflow
from airflow import DAG
from airflow.providers.apache.spark.operators.spark_submit import SparkSubmitOperator

def etl_pipeline():
    # Extract (ETL)
    extract_task = BashOperator(
        task_id='extract_data',
        bash_command="python3 /path/to/extract_script.py"
    )

    # Transform (ETL)
    transform_task = SparkSubmitOperator(
        task_id='transform_data',
        application='/path/to/transform_job.py',  # Python script for transformation
        conf={"spark.master": "local[*]"},
        jvm_options=["-Xmx1024M"]
    )

    # Load (ETL)
    load_task = BashOperator(
        task_id='load_data',
        bash_command="python3 /path/to/load_script.py"
    )

    extract_task >> transform_task >> load_task

def elt_pipeline():
    # Extract (ELT)
    extract_task = BashOperator(
        task_id='extract_data',
        bash_command="aws s3 cp s3://data-raw/ customer_transactions.csv"
    )

    # Transform & Load (ELT)
    process_and_load_task = SparkSubmitOperator(
        task_id='process_and_load_data',
        application='/path/to/process_and_load_job.py',  # Python script for processing and loading
        conf={"spark.master": "local[*]"},
        jvm_options=["-Xmx1024M"]
    )

    extract_task >> process_and_load_task
```

**Example input / output:**

```text
Input (ETL):
1. Extract raw customer transaction logs from multiple retailers.
2. Transform data to aggregate sales by date and time.

Output (ETL):
Load transformed aggregated data into the data warehouse for further analysis.

Input (ELT):
1. Extract raw customer transaction logs from multiple retailers and store them in S3.
2. Process data using Apache Spark, applying necessary transformations before loading back into the data warehouse.

Output (ELT):
Store processed data directly in the data warehouse after transformation.
```

**Data:**

```json
{
  "raw_data_sources": [
    "s3://data-raw/",
    "/path/to/extract_script.py",
    "/path/to/load_script.py"
  ],
  "processed_data_targets": [
    "/path/to/process_job_output/"
  ]
}
```

**Flow:**

1. Extract
2. ETL/ELT
3. Transform
4. Load ETL Data Warehouse
5. Process & Load ELT
6. Raw Data Sources
7. Fine-grained Control overTransformations
8. Extract
9. BatchProcessing/Transformation
10. ETL Load Processed Data
11. Transform in ETL
12. Load in ETL Data Warehouse
13. Transform in ELT
14. Load and Process in ELT

