---
type: "practice"
difficulty: "Deep"
---

# How do you handle a pipeline that must join telemetry across two clouds for a hybrid deployment while controlling egress cost and latency?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Orchestration vs Processing](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Orchestration_vs_Processing/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of joining telemetry data from two clouds in a hybrid deployment, we need to create an efficient and cost-effective data pipeline using tools like Airflow or Spark. The key is to balance egress costs and latency while ensuring high availability and reliability.

The question probes your understanding of distributed systems, network constraints, cost optimization techniques, and the use of ETL/ELT processes. We will assume that we are working with a typical hybrid cloud environment where data must be joined in real-time or near-real-time for analysis purposes.

Step-by-Step Solution
Data Sources: Identify the two clouds from which telemetry data is being sourced.
Data Synchronization: Ensure both data sources are synchronized and have consistent schemas, which may require schema normalization.
Network Constraints: Understand network latency between the two cloud environments, as it will impact egress costs and real-time processing capabilities.
Cost Optimization: Minimize egress costs by optimizing data transfer methods (e.g., using S3 transfers for less critical data).
Latency Management: Use in-memory storage or caching techniques to reduce latency during the join operation.
Scalability and Reliability: Ensure the pipeline can scale horizontally to handle varying loads and failures.

#### First-Principles Solution

Data Synchronization:

Utilize a continuous synchronization process using tools like AWS Data Pipeline, Azure Data Factory, or custom scripts to ensure both data sources are up-to-date.
Use consistent timestamps and versioning in the schemas of both clouds to avoid mismatches.

Network Latency Consideration:

Prioritize data that requires real-time processing over historical data.
Implement network optimization techniques such as CDN (Content Delivery Network) for faster data transfer between clouds.

Cost Optimization:

Use cost-effective storage options like S3 for less critical telemetry data to minimize egress costs.
Employ batch processing methods with ETL/ELT frameworks, and schedule tasks during off-peak hours when possible.
Implement data compression techniques to reduce the amount of data transferred.

Latency Management:

Utilize in-memory caching technologies like Redis or Memcached for frequently accessed data.
Employ Spark's in-memory processing capabilities to join datasets quickly, reducing latency.

Scalability and Reliability:

Design the pipeline using a microservices architecture where each component can scale independently.
Implement redundancy through failover mechanisms and retries with exponential backoff.
Monitor system performance and adjust as needed based on real-time metrics.

#### Closing Summary

By balancing network constraints, cost optimization techniques, and latency management strategies, we can create an efficient data pipeline for joining telemetry across two clouds. The key is to leverage tools like Airflow or Spark to manage the ETL/ELT processes while ensuring high availability and reliability. Common pitfalls include underestimating network latency, ignoring cost optimization, and not considering the scalability of the solution.

#### Common Mistakes/Follow-Ups
Ignoring Network Latency: Failing to account for significant network delays can lead to suboptimal performance.
Over-Egressing Data: Not optimizing egress costs by using efficient storage methods or scheduling data transfers during off-peak hours.
Insufficient Scalability Planning: Not designing the pipeline to handle varying loads and failures, leading to downtime.

**Code:**

```python
# Example Airflow DAG for joining telemetry across two clouds

from airflow import DAG
from airflow.operators.python_operator import PythonOperator
from datetime import timedelta

default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': datetime(2023, 1, 1),
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

dag = DAG('telemetry_join', default_args=default_args, schedule_interval=timedelta(hours=1))

def join_data():
    # Example code to join data from two clouds
    pass

join_task = PythonOperator(
    task_id='join_data',
    python_callable=join_data,
    dag=dag)

# Additional tasks for synchronization and cost optimization can be added here.
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Cloud1 Data (AWS S3) and Cloud2 Data (Azure Blob Storage) | Joined Telemetry Data in Real-Time or Near-Real-Time |

**Data:**

```json
{
  "cloud1": {
    "data_source": "AWS S3",
    "schema": ["timestamp", "device_id", "sensor_value"]
  },
  "cloud2": {
    "data_source": "Azure Blob Storage",
    "schema": ["timestamp", "device_id", "sensor_value"]
  }
}
```

