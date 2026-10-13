---
type: "practice"
difficulty: "Expert"
---

# A domain team wants to own its telemetry pipeline end-to-end in a mesh but lacks data-engineering skill, and a broken pipeline blinds on-call. How do you resolve decentralized ownership against uneven maturity, and what must the paved observability road provide for free?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Centralized vs Mesh Observability Data](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Centralized_vs_Mesh_Observability_Data.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of decentralized ownership and uneven maturity in a telemetry pipeline, we need to establish a robust data architecture that ensures end-to-end observability while providing a seamless experience for the domain team. The key is to leverage a mesh-based approach with structured governance and automation. This solution will ensure that the domain team can own their telemetry pipeline but still benefit from centralized controls.

#### Triaging the Question

The question revolves around balancing decentralized ownership (domain team responsibility) with centralized data engineering skills and robustness in the pipeline. The core challenge is to design a system where the domain team has autonomy over their telemetry data while mitigating risks such as broken pipelines that can impact on-call responsiveness.

Key Assumptions:
The domain team lacks internal data-engineering expertise.
A broken pipeline impacts on-call engineers, leading to operational disruptions.
The goal is to implement a solution that minimizes the risk of downtime without overburdening the domain team with complex setup and maintenance tasks.

#### First-Principles Solution
Decentralized Ownership: Empower the domain team by giving them control over their telemetry pipeline from collection through processing, storage, and visualization.
Central Governance: Establish a centralized governance framework that provides policy enforcement, monitoring, and support for the domain team to ensure data quality and compliance.
Automated Workflows: Implement automated processes to handle common tasks such as data ingestion, validation, and alerting, reducing the burden on the domain team.
Detailed Steps

Data Ingestion Layer:

Use a microservices architecture for collecting telemetry data from various sources (e.g., logs, metrics) into a distributed system.
Leverage open-source tools like Fluent Bit or Prometheus for lightweight data collection and forwarding.

Data Mesh Framework:

Design a data mesh where each domain owns its telemetry data, ensuring autonomy while maintaining consistency through shared services.
Implement a mesh of microservices that can be independently deployed by the domain team.

Centralized Data Warehouse:

Set up a centralized data warehouse (e.g., AWS Glue, Google BigQuery) to store and process aggregated telemetry data for analytics and reporting.
Use a governed ETL pipeline (e.g., Apache Airflow) to automate data transformations, ensuring consistent and high-quality data storage.

Monitoring and Alerting:

Implement continuous monitoring using tools like Grafana or Prometheus for real-time metrics visualization.
Configure alerting rules based on predefined thresholds and patterns to notify on-call engineers proactively.

Documentation and Training:

Provide comprehensive documentation and training resources to help the domain team understand their roles and responsibilities in maintaining the telemetry pipeline.
Offer regular support sessions and workshops to ensure the team is up-to-date with best practices.

#### Closing Summary

The solution involves creating a data mesh architecture where each domain has ownership of its telemetry pipeline while benefiting from centralized governance, monitoring, and automation. This approach ensures that the domain team can focus on their core business goals without being overwhelmed by complex setup and maintenance tasks.

#### Common Mistakes/Follow-Ups
Over-complicating the solution: Avoid unnecessary complexity that could overwhelm the domain team.
Neglecting governance: Ensure robust governance mechanisms are in place to maintain data quality and compliance.
Ignoring monitoring: Failing to set up adequate monitoring can lead to undetected issues during critical times.
Triaging Follow-Ups
Technical Debt:
How will the domain team handle technical debt without dedicated engineering resources?
Security Concerns:
What security measures are in place to protect sensitive telemetry data, especially when it's shared across domains?

**Comparison:**

| Domain | Source Types | Destination |
|--------|--------------|-------------|
| Sales  | Log Files, Metrics | Data Warehouse |
| Engineering | System Logs, Performance Metrics | Data Warehouse |

**Code:**

```python
# Example ETL Job using Apache Airflow
from airflow import DAG
from airflow.providers.apache.spark.operators.spark_submit import SparkSubmitOperator

default_args = {
    'owner': 'airflow',
    'start_date': datetime(2021, 1, 1),
}

dag = DAG(
    'telemetry_pipeline_dag',
    default_args=default_args,
    schedule_interval="@daily",
)

ingest_data_task = SparkSubmitOperator(
    task_id='ingest_data',
    application='/path/to/ingestion_script.py',
    conn_id='spark_default',
    dag=dag,
)
```

**Example input / output:**

```text
Input:
{
  "sales_data": [
    {"timestamp": "2023-10-01T12:00:00Z", "log_message": "Failed login attempt"},
    {"timestamp": "2023-10-01T14:00:00Z", "metric_value": 50}
  ],
  "engineering_data": [
    {"timestamp": "2023-10-01T12:00:00Z", "log_message": "Disk usage warning"},
    {"timestamp": "2023-10-01T14:00:00Z", "metric_value": 75}
  ]
}

Output:
{
  "sales_log_events": [
    {"timestamp": "2023-10-01T12:00:00Z", "message": "Failed login attempt"},
    {"timestamp": "2023-10-01T14:00:00Z", "metric_value": 50}
  ],
  "engineering_alerts": [
    {"timestamp": "2023-10-01T12:00:00Z", "message": "Disk usage warning"},
    {"timestamp": "2023-10-01T14:00:00Z", "metric_value": 75}
  ]
}
```

**Data:**

```json
{
  "domains": [
    {
      "name": "Sales",
      "pipeline": {
        "source": ["log_files", "metrics"],
        "destination": "data_warehouse"
      }
    },
    {
      "name": "Engineering",
      "pipeline": {
        "source": ["system_logs", "performance_metrics"],
        "destination": "data_warehouse"
      }
    }
  ]
}
```

**Flow:**

1. Data Ingestion
2. (Mesh of Microservices)
3. Domain-specific Pipelines
4. Centralized Data Warehouse
5. Monitoring & Alerting
6. On-call Notifications

