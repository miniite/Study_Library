---
type: "practice"
difficulty: "Deep"
---

# You need to backfill six months of telemetry into a new lake table format. Walk me through doing it without disrupting live ingestion or dashboards.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Hot vs Cold Telemetry](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Hot_vs_Cold_Telemetry.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To backfill six months of telemetry data into a new lake table format without disrupting live ingestion or dashboards, we need to carefully plan and execute a series of steps that ensure minimal downtime and accurate data integration. At this level, the focus is on ensuring robustness, minimizing disruptions, and maintaining high observability.

The primary challenge here is to handle large volumes of historical data while keeping the current pipeline running smoothly. We'll start by assessing our current system's architecture, understanding how telemetry is ingested, stored, and processed, then plan a step-by-step approach that includes data validation, transformation, and ingestion into the new format without breaking existing processes.

First, let’s break down the problem:

Current System Understanding: Identify where telemetry data lands, how it's validated, transformed, and stored.
Data Validation & Transformation: Develop a plan to validate historical data and transform it according to the new schema.
Parallel Ingestion: Ensure that both old and new formats coexist for a transition period without disrupting live ingestion or dashboards.
Testing: Perform thorough testing on a subset of data before full deployment.
Deployment & Monitoring: Gradually migrate data, monitor for any issues, and ensure smooth transitions.

Let's dive into each step in detail:

Step 1: Current System Understanding

Assuming our current architecture involves Kafka for real-time ingestion, Spark for transformation, and an S3 lake as storage, the first task is to map out this existing setup. We need to understand how data flows through the pipeline, from initial collection to final storage.

Step 2: Data Validation & Transformation

Historical telemetry data needs to be validated against the new schema. This can involve parsing JSON payloads, ensuring field consistency, and handling any potential discrepancies. Once validated, we'll transform the data according to the new format requirements.

To do this efficiently:

Use Apache Nifi or similar ETL tools for validation and transformation.
Schema Registry: Utilize a tool like Confluent Schema Registry to manage schema evolution, ensuring that historical data aligns with the new schema without breaking existing pipelines.
Step 3: Parallel Ingestion

We'll need to set up a parallel ingestion mechanism where both old and new formats coexist. This requires:

Staged Storage: Introduce an intermediate layer where transformed data is stored before being merged back into the main lake.
Batch Jobs: Schedule periodic batch jobs using Apache Airflow or similar tools to ingest historical data, ensuring minimal impact on live ingestion.
Step 4: Testing

Before full deployment, conduct thorough testing with a subset of historical data:

Unit Tests: Validate individual components and transformations.
Integration Tests: Simulate end-to-end processes to ensure all parts work together seamlessly.
Performance Testing: Measure the performance impacts during backfilling to avoid bottlenecks.
Step 5: Deployment & Monitoring

Finally, gradually deploy the new schema and validate its behavior:

Rollout Plan: Start with a small portion of historical data to test the entire pipeline.
Monitoring: Set up comprehensive monitoring using tools like Prometheus or Grafana to track performance metrics and any anomalies post-deployment.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, the key steps are:

Understanding the current system architecture.
Validating and transforming historical data using ETL tools and schema registries.
Setting up parallel ingestion with staged storage.
Thorough testing to ensure robustness.
Gradual deployment and monitoring.

#### Common mistakes
include failing to validate historical data properly, not setting up adequate staging layers, and ignoring performance impacts during backfilling. Follow-ups could involve discussing specific tools used (e.g., Nifi, Airflow) or delving into more detailed architecture decisions.

**Comparison:**

```
Transformation Step	Description
Validation	Ensuring historical data aligns with new schema.
Transformation	Converting old data structures into new format.
Storage	Saving transformed data in the lake table.
```

**Code:**

```python
# Example of a simple transformation step in Python using pandas
import pandas as pd

def transform_data(input_df):
    # Validate and transform data
    transformed_df = input_df  # Placeholder for actual transformation logic
    return transformed_df

# Example Airflow task for backfilling historical data
from airflow.operators.python_operator import PythonOperator

def backfill_task(ti):
    # Retrieve historical data from S3
    raw_data = get_historical_data_from_s3()

    # Transform and validate data
    transformed_data = transform_data(raw_data)

    # Store in the lake table
    store_in_lake(transformed_data)

# Schedule a task for backfilling
backfill_task = PythonOperator(
    task_id='backfill_task',
    python_callable=backfill_task,
    dag=dag
)
```

**Example input / output:**

```text
**Input**: Historical telemetry data in JSON format.

**Output**: Data validated and transformed according to the new schema, stored in the lake table.
```

**Data:**

```json
{
  "schema": {
    "fields": [
      {"name": "timestamp", "type": "string"},
      {"name": "metrics", "type": "object"}
    ]
  }
}
```

**Flow:**

1. Historical Data Collection
2. Validate & Transform
3. Staged Storage
4. Parallel Ingestion
5. Lake Table

