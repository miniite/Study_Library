---
type: "practice"
difficulty: "Deep"
---

# How would you schedule and tune Iceberg table maintenance (expire snapshots rewrite manifests compaction) in Airflow so table read performance doesn't degrade over months?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Schema Drift Handling](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Schema_Drift_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that an Iceberg table's read performance does not degrade over time, we need to manage the expiration of snapshots, rewrite manifests, and perform compactions in a way that is both efficient and minimizes performance impact on reads. This process involves scheduling tasks in Apache Airflow to handle these operations while optimizing resource usage and considering the trade-offs between maintenance overhead and read performance.

#### Triaging the Question

The core question revolves around scheduling and tuning Iceberg table maintenance tasks (snapshot expiration, manifest rewriting, compaction) within an Airflow DAG. The goal is to balance the frequency of these tasks with their impact on read performance. We need to understand how each operation affects the system, set appropriate intervals for triggering them, and ensure that resources are managed effectively.

#### First-Principles Solution
Snapshot Expiration: Iceberg keeps multiple snapshots of table states, which can be large if not regularly cleaned up. Expiring old snapshots reduces storage overhead but should not happen too frequently to avoid disrupting ongoing reads.
Manifest Rewriting: Manifest files are crucial for reading data efficiently. Regularly rewriting them ensures that the latest state is reflected and optimizes read performance. This process should be scheduled more often than snapshot expiration.
Compaction: Compactions consolidate small files into larger ones, reducing storage overhead but increasing write latency and read performance during the compaction period.
Detailed Steps

Identify Critical Operations:

Snapshot Expiration: Typically every 2-4 weeks.
Manifest Rewriting: Daily or hourly to keep up with frequent writes.
Compaction: Less frequently, perhaps monthly or quarterly, depending on write activity and performance metrics.

Airflow DAG Design:

Create separate tasks for each operation using Airflow's BashOperator or PythonOperator.
Use dynamic parameters in task definitions to conditionally execute based on the state of the Iceberg table.
Set dependencies between tasks to ensure compaction happens after manifest rewriting and snapshot expiration.

Resource Management:

Configure Airflow to use dedicated resources for these tasks, especially during compactions which are resource-intensive.
Monitor task performance and adjust intervals or resource allocation as needed.

Monitoring and Feedback Loop:

Implement logging and metrics collection using tools like Prometheus + Grafana to monitor table performance and identify bottlenecks.
Use this data to fine-tune the scheduling and resource allocation over time.

#### Closing Summary

By carefully managing snapshot expiration, manifest rewriting, and compaction through a well-architected Airflow DAG, we can maintain high read performance for an Iceberg table without compromising on data integrity or storage efficiency. The key is to balance these operations based on real-world usage patterns and continuously refine the process.

#### Common Mistakes
and Follow-Ups
Overzealous Compaction: Too frequent compactions can degrade write performance, leading to overall system slowdown.
Insufficient Snapshot Expiration: Retaining too many old snapshots can lead to excessive storage costs and slower read operations.
Inadequate Monitoring: Lack of monitoring can result in missed issues or incorrect tuning decisions.

#### Follow-ups
might include questions about specific Airflow configurations, integration with other tools like Spark for compaction, or handling data lineage and governance requirements.

**Code:**

```python
# Example Airflow DAG for Iceberg Table Maintenance
from datetime import timedelta
from airflow import DAG
from airflow.operators.python_operator import PythonOperator

default_args = {
    'owner': 'data_engineer',
    'depends_on_past': False,
    'start_date': '2023-01-01',
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

dag = DAG(
    'iceberg_table_maintenance',
    default_args=default_args,
    description='Maintain Iceberg table performance over time.',
    schedule_interval='@daily',  # Adjust based on workload
)

def expire_snapshots():
    # Logic to expire old snapshots
    pass

def rewrite_manifests():
    # Logic to rewrite manifests
    pass

def perform_compaction():
    # Logic to initiate compaction
    pass

# Task Definitions
expire_task = PythonOperator(
    task_id='expire_snapshots',
    python_callable=expire_snapshots,
    dag=dag,
)

rewrite_task = PythonOperator(
    task_id='rewrite_manifests',
    python_callable=rewrite_manifests,
    dag=dag,
)

compaction_task = PythonOperator(
    task_id='perform_compaction',
    python_callable=perform_compaction,
    dag=dag,
)

# Task Dependencies
expire_task >> rewrite_task >> compaction_task
```

**Flow:**

1. Expire Snapshots
2. Rewrite Manifests
3. Perform Compaction
4. Monitor Performance

