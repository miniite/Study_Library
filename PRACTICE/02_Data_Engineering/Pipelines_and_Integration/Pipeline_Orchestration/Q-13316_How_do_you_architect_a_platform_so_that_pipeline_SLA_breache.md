---
type: "practice"
difficulty: "Deep"
---

# How do you architect a platform so that pipeline SLA breaches auto-escalate and auto-remediate (failover backfill reroute) before a human is paged?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Shared Pipeline Platforms](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Orchestration_vs_Processing/Shared_Pipeline_Platforms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of automating the escalation and remediation process in data pipelines, we need to architect a platform that can proactively monitor pipeline performance, trigger automatic interventions, and escalate issues to human operators only when necessary. The key is to design a system with robust monitoring, alerting, and failover mechanisms, leveraging tools like Apache Airflow or Apache Spark for the core processing.

This solution involves several steps: setting up reliable monitoring, configuring automated alerts, implementing failover strategies, and ensuring seamless integration between monitoring and remediation systems. We'll start by defining our key requirements and then break down the architecture into manageable components.

**Comparison:**

```
Escalation Level	Action Taken	Email Content	Operator Paged?
1	Reroute data to backup	Data pipeline processing time exceeded 300s; rerouting in progress.	No
2	Failover primary to backup	Data pipeline error rate exceeded 10%; failover initiated, critical alerts sent.	No
3+	Human intervention required	Manual intervention needed due to ongoing issues post-failover.	Yes
```

**Code:**

```python
# Example Airflow DAG for a simple data pipeline
from datetime import timedelta
from airflow import DAG
from airflow.operators.python_operator import PythonOperator

default_args = {
    'owner': 'data_engineer',
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
}

dag = DAG(
    'example_data_pipeline_dag',
    default_args=default_args,
    description='A simple example of an Airflow data pipeline',
    schedule_interval=timedelta(hours=1),  # Run every hour
)

def check_pipeline_health():
    # Placeholder for health check logic
    return "Healthy"

check_pipeline = PythonOperator(
    task_id='check_pipeline',
    python_callable=check_pipeline_health,
    dag=dag,
)
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Pipeline reports processing time of 320 seconds and error rate of 15% (both exceeding thresholds) at 9 AM. | System triggers failover to backup pipeline and sends escalation email. Human operator is paged only if the issue persists after failover. |

**Data:**

```json
{
  "monitoring": {
    "metrics": ["processing_time", "error_rate"],
    "thresholds": {"processing_time": 300, "error_rate": 10}
  },
  "remediation": {
    "actions": ["reroute_data", "failover"],
    "escalation_levels": [5, 10, 20]
  }
}
```

**Flow:**

1. Yes
2. No
3. No
4. Yes
5. Pipeline starts
6. Is processing time >threshold?
7. Reroute data
8. Is error rate > threshold?
9. Failover to backup pipeline
10. Send escalation email
11. Monitor recovery
12. Pipeline is healthy
13. Human operator paged forintervention

