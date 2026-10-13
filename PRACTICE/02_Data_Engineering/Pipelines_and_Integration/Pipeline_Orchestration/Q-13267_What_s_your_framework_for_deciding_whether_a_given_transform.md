---
type: "practice"
difficulty: "Deep"
---

# What's your framework for deciding whether a given transform belongs in the warehouse (dbt/SQL) the lakehouse (Spark) or streaming (Flink) as an org-level standard rather than per-team preference?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Choosing Processing Engines](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Batch_vs_Streaming_Features/Choosing_Processing_Engines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding where to place a data transformation process—whether in the warehouse, lakehouse, or streaming environment—the key is understanding the lifecycle and requirements of your data. A robust framework should consider factors such as data freshness, compute resources, storage costs, and operational complexity. Here’s my approach:

#### Triaging the Question
Data Freshness: Determine if real-time updates are critical.
Compute Resources: Assess whether batch processing or low-latency streaming is required.
Storage Costs: Evaluate long-term storage needs.
Operational Complexity: Consider ease of deployment, monitoring, and maintenance.
Core Idea

The goal is to create a scalable and maintainable data infrastructure that aligns with business objectives while balancing technical constraints.

Step-by-Step Solution
Define the Data Lifecycle:
Batch Processing (Warehouse/Warehouse-First): Use dbt or SQL for transformations that occur infrequently, such as nightly batch jobs.
Incremental Updates (Lakehouse/Stream-Processing): Leverage Spark and Flink for real-time updates and incremental loads.
Evaluate Data Requirements:
Real-Time vs Batch: If the data transformation needs to be applied in near-real time, consider using streaming frameworks like Flink or Spark Streaming.
Latency Tolerance: For transformations that can tolerate some latency, batch processing through dbt is sufficient.
Resource Utilization:
Compute Resources: Batch jobs often require less complex and resource-light environments compared to continuous streaming applications.
Storage Costs: Determine the storage requirements for each step in the pipeline to avoid unnecessary costs.
Operational Considerations:
Deployment Complexity: Stream processing can be more complex to set up due to fault tolerance, checkpointing, and state management.
Monitoring and Maintenance: Ensure robust monitoring tools are available for both batch and streaming environments.
Detailed Solution

Batch Processing (dbt/SQL):

Use Case: Reporting, analytics, data warehousing.
Execution: Scheduled jobs running nightly or at specified intervals.

#### Example
A daily ETL process that populates the warehouse using dbt models.

Incremental Updates (Spark/Flink):

Use Case: Real-time analytics, fraud detection, user behavior tracking.
Execution: Continuous streaming processing with low latency requirements.

#### Example
A Flink job that processes incoming logs and updates a real-time dashboard.

Architectural Considerations:

Data Lakehouse (Delta Lake, Iceberg): Store raw data efficiently for both batch and stream transformations.
Integration Layer: Use Apache Airflow to manage workflows across different processing layers.

#### Closing Summary

In summary, the decision on where to place a data transformation process depends on the specific needs of the data lifecycle. Batch processing is ideal for reporting and analytics, while real-time transformations require streaming frameworks like Spark or Flink. The organization should establish clear guidelines based on these considerations to ensure consistency across teams.

#### Common Mistakes/Follow-Ups
Overlooking Real-Time Needs: Underestimating the importance of real-time insights can lead to missed business opportunities.
Ignoring Storage Costs: Poor planning for storage costs can result in inefficient use of resources and increased expenses.
Overcomplicating Batch Processes: Failing to leverage batch processing tools like dbt can lead to unnecessary complexity.

**Comparison:**

| Data Type | Use Case           | Framework   | Example Pipeline       |
|-----------|--------------------|-------------|-----------------------|
| Batch     | Reporting          | dbt/SQL     | Daily ETL Job         |
| Incremental | Real-Time Analytics | Spark/Flink | Real-time Stream Processing |

**Code:**

```
Example dbt model for batch processing

from dbt.utils import log

def calculate_metrics(data): # Perform calculations metrics = {} log.info("Calculating daily metrics") return metrics

model_name = "daily_metrics"

CODE
Copy

```python
# Example Flink job for real-time processing
import pyflink.dataset.table as t_env

table_env = t_env/TableEnvironment()
source_table = table_env.from_path("/path/to/source")
sink_table = table_env.from_path("/path/to/sink")

def process_data(row):
    # Process and emit data
    result = {}
    return result

result_table = source_table.map_element(process_data)
result_table.insert_into(sink_table)

table_env.execute("Real Time Processing Job")
```

**Example input / output:**

```text
Input (Batch Processing):
- Raw Data: {"timestamp": "2023-10-01T12:00:00Z", "value": 100}

Output (Batch Processing):
- Transformed Data: {"timestamp": "2023-10-01T12:00:00Z", "metric1": 120, "metric2": 90}

Input (Real-Time Processing):
- Raw Stream: [{"timestamp": "2023-10-01T12:05:00Z", "value": 110}]

Output (Real-Time Processing):
- Transformed Stream: {"timestamp": "2023-10-01T12:05:00Z", "metric1": 130, "metric2": 100}
```

**Data:**

```json
{
  "raw_data": [
    {"timestamp": "2023-10-01T12:00:00Z", "value": 100},
    {"timestamp": "2023-10-01T12:05:00Z", "value": 110}
  ],
  "transformed_data": [
    {"timestamp": "2023-10-01T12:00:00Z", "metric1": 120, "metric2": 90},
    {"timestamp": "2023-10-01T12:05:00Z", "metric1": 130, "metric2": 100}
  ]
}
```

**Flow:**

1. Data Source
2. dbt/SQL
3. Spark/Flink
4. Warehouse
5. Lakehouse
6. Dashboard
7. Lakehouse Storage

