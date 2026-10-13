---
type: "practice"
difficulty: "Deep"
---

# An Airflow-triggered Databricks job occasionally double-runs because the trigger retried. How do you make the whole ingest idempotent end-to-end?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Streaming Ingest Tuning](<../../../../TOOLS/Databricks/Databricks_Platform/Streaming_Ingest_Tuning.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of an Airflow-triggered Databricks job double-running due to a retry, we need to ensure that the entire data ingestion process is idempotent. Idempotency means that the same operation can be applied multiple times without changing the outcome after the first application. This involves both ensuring that the Databricks job itself handles duplicates correctly and ensuring that the trigger logic in Airflow avoids unnecessary retries.

#### Triaging the Question

The core of the question is about making an ingestion process idempotent end-to-end, which means we need to ensure no data duplication during processing. The key aspects to consider are:

Airflow Retry Mechanism: Understanding how retries work and their potential impact on Databricks jobs.
Databricks Job Handling: Ensuring the job logic can handle duplicate records without producing incorrect results.

#### Assumptions
We assume that the data ingestion process is triggered by Airflow, which may retry a job if it fails for some reason (e.g., network issues).
The Databricks job processes this data and writes output back to a storage system.
The goal is to avoid writing duplicate records into the final destination.

#### First-Principles Solution

To make the entire process idempotent, we need to ensure that:

Airflow Job Retries Are Controlled: Airflow retries should be handled in such a way that they do not lead to multiple executions of the Databricks job.
Databricks Job Logic Ensures Idempotency: The Databricks job must handle duplicate records gracefully.
Step 1: Controlling Airflow Retries

Airflow provides mechanisms like retry_exponential_backoff and retries. We need to configure these settings to avoid unnecessary retries that could trigger the Databricks job multiple times. For example, we can set a maximum number of retries:

PYTHON
default_args = {
    'owner': 'data_engineer',
    'depends_on_past': False,
    'start_date': datetime(2023, 1, 1),
    'retries': 2,
    'retry_delay': timedelta(minutes=5),
}

Additionally, we can use sla_miss to handle delayed execution cases where a job should not be retried if it misses its SLA:

PYTHON
default_args[' sla_mins'] = [timedelta(minutes=30)]
Step 2: Making the Databricks Job Idempotent

To ensure the Databricks job is idempotent, we need to implement logic that checks for existing records before inserting new ones. This can be done using upsert operations (insert or update based on a unique key).

Here’s an example of how you might structure this in Python within a Databricks notebook:

PYTHON
from pyspark.sql import SparkSession

spark = SparkSession.builder.getOrCreate()

# Read the incoming data
data = spark.read.format('csv').option('header', 'true').load('/path/to/input')

# Define a unique key for upsert operations
unique_key = "record_id"

# Load existing records from storage (e.g., Delta Lake)
existing_data = spark.read.table('existing_table')

# Perform an upsert operation to handle duplicates
data.join(existing_data, unique_key, 'leftanti').write.mode('append').saveAsTable('output_table')

In this example:

We read the incoming data and define a unique_key that identifies each record.
We load existing records from storage.
We use a left anti join to find records in the new batch that are not already present in the existing data.

#### Closing Summary

To make an Airflow-triggered Databricks job idempotent end-to-end, we need to control the retries at the Airflow level and ensure that the Databricks job logic can handle duplicate records gracefully. This involves configuring Airflow’s retry mechanism carefully and implementing upsert operations in the Databricks job.

#### Common Mistakes/Follow-Ups
Incorrect Retry Configuration: Not setting appropriate retry limits or misconfiguring SLAs.
Inadequate Idempotency Logic: Failing to implement proper checks for existing records, leading to duplicate entries.
Lack of Monitoring and Logging: Not having robust monitoring in place to detect and handle retries.

If asked about implementing this in a production environment, I would also recommend setting up detailed logging and alerting mechanisms to track the state of the data processing pipeline.

**Comparison:**

```
Yes
No
Input Data
Check for Existing Record
No Change
Insert New Record
Output Table
Output Table
```

**Code:**

```python
# Airflow retry configuration example
default_args = {
    'owner': 'data_engineer',
    'depends_on_past': False,
    'start_date': datetime(2023, 1, 1),
    'retries': 2,
    'retry_delay': timedelta(minutes=5),
}

# Databricks job logic for idempotency
from pyspark.sql import SparkSession

spark = SparkSession.builder.getOrCreate()

data = spark.read.format('csv').option('header', 'true').load('/path/to/input')
unique_key = "record_id"
existing_data = spark.read.table('existing_table')

# Perform upsert operation
data.join(existing_data, unique_key, 'leftanti').write.mode('append').saveAsTable('output_table')
```

**Example input / output:**

| Input Data             | Existing Table         | Output Table           |
|------------------------|------------------------|------------------------|
| ID: 1, Value: record1   | ID: 1, Value: record1  | No change              |
| ID: 2, Value: record2   | None                   | Inserted               |
| ID: 3, Value: record3   | None                   | Inserted               |

**Data:**

```json
{
  "input": [
    {
      "id": 1,
      "value": "record1"
    },
    {
      "id": 2,
      "value": "record2"
    },
    {
      "id": 3,
      "value": "record3"
    }
  ],
  "existing_data": [
    {
      "id": 1,
      "value": "record1"
    },
    {
      "id": 4,
      "value": "record4"
    }
  ]
}
```

**Flow:**

1. Start
2. Read Input Data
3. Determine Unique Key
4. Load Existing Data
5. Perform Left Anti Join
6. Write to Output Table

