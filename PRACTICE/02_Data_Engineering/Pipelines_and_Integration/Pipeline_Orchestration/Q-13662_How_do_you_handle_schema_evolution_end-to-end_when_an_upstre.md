---
type: "practice"
difficulty: "Expert"
---

# How do you handle schema evolution end-to-end when an upstream Avro schema in the Kafka registry adds a field your Spark streaming job and Iceberg sink must both absorb?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Schema Drift Handling](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Schema_Drift_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address schema evolution in data pipelines, particularly when an upstream Avro schema changes and impacts downstream processing (Spark streaming and Iceberg), we need to ensure that our system can handle these changes seamlessly without downtime or data loss. This involves a combination of schema registry management, versioning strategies, and robust pipeline design.

First, let's restate the problem: An upstream Avro schema in Kafka has been updated with an additional field, and this change needs to be propagated to both the Spark streaming job and the Iceberg sink. The key aspects we need to consider are how to identify the new schema version, validate compatibility, and update our data processing logic accordingly.

#### Triaging the Question

The question is essentially asking us to design a robust mechanism for handling schema evolution in a data pipeline using Apache Airflow for orchestration and Spark streaming for real-time processing. We must ensure that the changes are propagated correctly and that both the downstream systems (Spark and Iceberg) can handle these changes without breaking.

#### Assumptions
The Kafka topic has an associated Avro schema stored in a registry like Confluent Schema Registry.
Our Spark streaming job reads from this Kafka topic.
We use Apache Airflow to manage our data pipeline jobs, ensuring that the pipeline is reconfigured when needed.
Iceberg tables are used for efficient storage and querying of large datasets.
Step-by-Step Solution
Schema Versioning: Implement a versioned schema strategy in your Kafka registry. Each time a new field is added to the schema, increment its version number. This ensures that you can track which versions of the schema are being used.
Airflow Tasks for Schema Changes:
Create an Airflow task to periodically check the schema version in the registry.
If a new schema version is detected, mark this as a "schema change" event.
Schema Validation and Transformation:
Implement a schema validation step within the Spark streaming job that can handle both old and new schema versions.
Use a DataCleaner or similar library to automatically transform data from older schemas to newer ones if necessary.
Airflow DAG Retriggering:
Upon detecting a schema change, retrigger the relevant Airflow tasks to apply the updated logic.
Ensure that any downstream processing also picks up these changes.

#### First-Principles Solution

Schema Version Check: In the Airflow task responsible for reading from Kafka, include a step to check the latest schema version in the registry.

Validation and Transformation Logic:

PYTHON
# Example Python code snippet
def validate_and_transform_data(df, new_schema):
    if df.schema != new_schema:
        # Perform transformation logic here
        transformed_df = transform_df_with_new_schema(df, new_schema)
        return transformed_df
    else:
        return df

# Within the Airflow task
schema_version = check_latest_schema_version()
df = read_kafka_data()
updated_df = validate_and_transform_data(df, schema_version)
store_in_iceberg(updated_df)

Airflow Task Configuration:

Define an Airflow task to periodically check the schema version.
Use BranchPythonOperator or similar constructs to conditionally retrigger tasks if a schema change is detected.

#### Closing Summary

By implementing these strategies, we ensure that our data pipeline can handle schema evolution seamlessly. The key steps involve versioning schemas, validating and transforming data on-the-fly, and leveraging Airflow for automated task reconfiguration. This approach minimizes downtime and ensures that both the Spark streaming job and the Iceberg sink are always up-to-date with the latest schema versions.

#### Common Mistakes
and Follow-Ups
Ignoring Schema Versioning: Not properly versioning schemas can lead to data loss or corruption when new fields are added.
Manual Schema Management: Relying on manual updates can introduce human error and delays in updates.
Inefficient Transformation Logic: Improper transformation logic could slow down processing, especially with large datasets.

**Code:**

```python
from confluent_kafka.schema_registry import AvroSerializer, KafkaAvroDeserializer
import pyspark.sql.functions as F

def check_latest_schema_version():
    # Function to query the schema registry and get the latest version
    pass

def transform_df_with_new_schema(df, new_schema):
    # Transformation logic here
    return df

def validate_and_transform_data(df, new_schema):
    if df.schema != new_schema:
        transformed_df = transform_df_with_new_schema(df, new_schema)
        return transformed_df
    else:
        return df

schema_version = check_latest_schema_version()
df = read_kafka_data()
updated_df = validate_and_transform_data(df, schema_version)
store_in_iceberg(updated_df)
```

**Flow:**

1. Yes
2. No
3. Check Latest Schema Version
4. Is New Schema Present?
5. Update Airflow Task
6. Continue Processing withCurrent Schema
7. Trigger Reloading of SparkStreaming Job and IcebergWrite Operation

