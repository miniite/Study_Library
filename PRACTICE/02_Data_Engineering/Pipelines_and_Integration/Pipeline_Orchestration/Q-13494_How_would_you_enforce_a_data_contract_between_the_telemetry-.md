---
type: "practice"
difficulty: "Deep"
---

# How would you enforce a data contract between the telemetry-producer teams and your Spark anomaly pipeline so a breaking metric-label change can't silently poison detection?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Pipeline Data Monitoring](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Pipeline_Data_Monitoring.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that telemetry-producer teams adhere to a consistent data contract, we need to enforce strict validation rules in our Spark-based anomaly detection pipeline using Airflow orchestrations. This approach not only prevents breaking changes from silently corrupting detection but also ensures the integrity and consistency of the dataset.

#### Triaging the Question

The core issue here is maintaining data contract compliance between producer teams and the downstream anomaly detection pipeline. The question aims to understand how we can enforce a robust validation mechanism that checks for label consistency, ensuring no breaking change in metrics labels goes unnoticed. This involves understanding the data flow, identifying potential failure points, and designing a system that actively detects and prevents such issues.

#### First-Principles Solution
1. Data Model and Schema Validation

First, define a clear schema for the telemetry data produced by each team. This includes metadata like metric names, label keys, and their allowable values. Use this schema to validate incoming data in real-time or during batch processing stages.

2. Airflow Orchestration

Leverage Airflow to schedule periodic validation tasks that run alongside the anomaly detection pipeline. These tasks should act as a guardrail, ensuring that any deviation from the expected schema triggers alerts and fails the job if necessary.

3. Spark Data Validation Libraries

Use Spark’s built-in data validation libraries or third-party tools like PySpark Validation to implement these checks during ETL processes. This ensures that the data ingested by Spark is always in line with our predefined schemas before further processing occurs.

4. Proactive Monitoring and Alerting

Implement a robust monitoring system that tracks any schema changes reported by producer teams. Use this information to update validation rules and ensure they are applied consistently across all pipelines.

Detailed Steps

Define the Data Contract:

Create a JSON or YAML configuration file defining the expected schema for each metric.

#### Example
YAML
telemetry_schema:
  metrics:
    - name: "cpu_usage"
      labels:
        - key: "host_id"
          value_pattern: "[0-9]+"
        - key: "time"
          value_type: "timestamp"

Integrate Validation in Airflow:

PYTHON
from airflow.models import BaseOperator
from pyspark.sql.functions import col, when

class ValidateTelemetryData(BaseOperator):
    def __init__(self, schema_path, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.schema_path = schema_path

    def execute(self, context):
        # Load schema
        with open(self.schema_path) as f:
            schema = yaml.safe_load(f)

        # Fetch data from Spark DataFrame
        df = context['task_instance'].xcom_pull(task_ids='fetch_data')

        for metric_name, metric_details in schema['metrics'].items():
            for label_key, label_config in metric_details['labels'].items():
                validation_logic = f"col('{label_key}') != {label_config}"
                df = df.filter(~when(eval(validation_logic), True).otherwise(False))

        # If any row remains invalid, fail the job
        if not df.isEmpty():
            raise ValueError("Data validation failed")

Monitor Schema Changes:

Integrate a change management system that logs schema updates.
Use Git hooks or other version control tools to track changes.

Update Validation Rules Dynamically:

Implement an update mechanism where new schemas are automatically picked up and applied in the validation tasks without manual intervention.

#### Closing Summary

By combining Airflow orchestration with Spark data validation, we can ensure that telemetry producer teams adhere to a strict data contract. This approach not only prevents breaking changes but also provides proactive monitoring and failsafe mechanisms to catch issues early. The key is to design an automated system that enforces these rules at every stage of the pipeline.

#### Common Mistakes/Follow-Ups
Failing to Define Clear Schemas: Without a clear definition, it's hard to enforce any validation.
Not Integrating Real-Time Validation: Offline validation might miss real-time issues.
Lack of Automation for Schema Updates: Manually updating rules can lead to inconsistencies and delays.

Do you have any specific concerns or additional details about this setup?

**Comparison:**

| Metric Name | Label Key | Value Pattern     | Validation Logic                          |
|-------------|-----------|------------------|------------------------------------------|
| cpu_usage   | host_id   | [0-9]+           | col('host_id') != "[0-9]+"                |
|             | time      | timestamp        | col('time').isNotNull()                    |

**Code:**

```python
from airflow.models import BaseOperator
from pyspark.sql.functions import col, when

class ValidateTelemetryData(BaseOperator):
    def __init__(self, schema_path, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.schema_path = schema_path

    def execute(self, context):
        with open(self.schema_path) as f:
            schema = yaml.safe_load(f)

        df = context['task_instance'].xcom_pull(task_ids='fetch_data')

        for metric_name, metric_details in schema['metrics'].items():
            for label_key, label_config in metric_details['labels'].items():
                validation_logic = f"col('{label_key}') != {label_config}"
                df = df.filter(~when(eval(validation_logic), True).otherwise(False))

        if not df.isEmpty():
            raise ValueError("Data validation failed")
```

**Example input / output:**

```

Sample input DataFrame before validation:

PYTHON
Copy
+-----+--------+-------------------+
|name |label   |value              |
+-----+--------+-------------------+
|cpu_usage| host_id | 123               |
|cpu_usage| time    | 2023-10-01T09:00  |
|cpu_usage| host_id | abc               |
+-----+--------+-------------------+

Sample output DataFrame after validation:

PYTHON
Copy
+-----+--------+-------------------+
|name |label   |value              |
+-----+--------+-------------------+
|cpu_usage| time    | 2023-10-01T09:00  |
+-----+--------+-------------------+

The validation logic failed for host_id with value "abc", leading to an error.
```

**Data:**

```yaml
telemetry_schema:
  metrics:
    - name: "cpu_usage"
      labels:
        - key: "host_id"
          value_pattern: "[0-9]+"
        - key: "time"
          value_type: "timestamp"
```

**Flow:**

1. Yes
2. No
3. Fetch Data
4. Is Schema Valid?
5. Proceed with Validation
6. Validate Data
7. F
8. Update Schema & RetryValidation
9. Detect Anomalies

