---
type: "practice"
difficulty: "Deep"
---

# A data product you consume publishes a 99.9% freshness SLA but your model needs fresher data. How do you handle that mismatch without owning their pipeline?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Data Product SLAs](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Data_Product_SLAs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the mismatch between a data product's 99.9% freshness SLA and your model’s requirement for even more frequent updates, we need to implement an effective caching or pre-processing strategy that leverages existing data architectures like lakes, warehouses, and mesh architectures. The core idea is to create a robust, scalable solution that maximizes the utility of the published data while minimizing the latency between when the data product publishes it and when your model consumes it.

#### Triage
Question: How do you handle a 99.9% freshness SLA from an external data product without owning their pipeline?
Core Probing Points:
Understanding of data architecture principles (Lakes, Warehouses, Mesh).
Knowledge of caching strategies and their trade-offs.
Experience with real-time vs. batch processing.
Familiarity with the implications on system performance and cost.

#### Assumptions
The external data product publishes semi-regularly but not at a cadence that aligns perfectly with your model’s needs.
You have access to intermediate storage solutions (e.g., data warehouses, data lakes) for processing and caching.
Your system can tolerate some level of latency between the published data and its consumption by the model.

#### First-Principles Solution

Data Lake as Intermediate Storage:

#### Solution
Use a data lake to store the raw data from the external source before it's processed into a more usable form.
Why?: Data lakes are designed for raw, unprocessed data and can handle large volumes of semi-regularly published data.

ETL Pipelines:

#### Solution
Implement an ETL pipeline that pulls data from the external source and processes it to meet your model’s requirements.
Why?: This allows you to add value by cleaning, transforming, and enriching the data as needed before using it in your models.

Caching Mechanisms:

#### Solution
Use caching mechanisms such as Redis or Memcached to store frequently accessed subsets of the processed data.
Why?: Caching reduces the latency between data publication and model consumption, ensuring that your model always has access to the freshest possible data.

Data Mesh Architecture:

#### Solution
Integrate a data mesh architecture to ensure that multiple teams can consume and process the same datasets independently.
Why?: This approach allows for more flexibility in how different parts of the organization use the data, reducing dependencies on a single pipeline.

Real-Time Processing with Streaming Technologies:

#### Solution
Deploy real-time processing technologies like Apache Kafka or AWS Kinesis to handle incremental updates and ensure that your model can consume new data as it becomes available.
Why?: Real-time processing ensures that any changes in the external source are reflected immediately, aligning better with your model’s needs.

Cost Optimization:

#### Solution
Optimize costs by using tiered storage solutions where less frequently accessed data is stored at lower cost tiers.
Why?: Balancing storage costs and performance requirements to ensure that the system remains efficient even as it scales.

#### Closing Summary
+ Common Mistakes/Follow-Ups:
Summary: By leveraging a combination of data lake, ETL pipelines, caching mechanisms, real-time processing technologies, and cost optimization strategies, you can effectively handle the mismatch between an external data product's SLA and your model’s needs without owning their pipeline.

#### Common Mistakes
Overlooking the importance of intermediate storage solutions like data lakes.
Ignoring the need for caching mechanisms to reduce latency.
Failing to consider real-time processing technologies, which can be crucial for high-fidelity models.
Neglecting cost optimization strategies that could lead to unnecessary expenses.

#### Follow-Ups
Interviewer: How do you measure and ensure the quality of data in your system?
Interviewer: Can you describe a scenario where real-time processing might not be feasible, and how would you address it?

**Comparison:**

| Data Source | Storage Tier | Processing Pipeline | Caching Mechanism |
|-------------|--------------|---------------------|------------------|
| External    | Raw          | ETL                 | Redis            |
| Intermediate| Processed    | Transform           | Memcached        |
| Final       | Optimized    | Real-time           | None             |

**Code:**

```python
# Example ETL Pipeline using Apache Airflow for scheduling
from airflow import DAG
from airflow.operators.python_operator import PythonOperator

default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': datetime(2021, 1, 1),
    'email_on_failure': False,
    'email_on_retry': False,
}

dag = DAG('external_data_pipeline', default_args=default_args)

def process_data(**kwargs):
    # Example data processing logic
    pass

process_task = PythonOperator(
    task_id='process_external_data',
    python_callable=process_data,
    dag=dag,
)
```

**Example input / output:**

```text
Input: External data published every hour.
Output: Processed and cached data available for model consumption within 5 minutes of the latest publication.

Request: How to handle real-time updates?
Response: Use Apache Kafka or AWS Kinesis for real-time ingestion, ensuring that any new data is consumed immediately by your system.
```

**Data:**

```yaml
# Sample Data Schema for Data Lake Storage
data_lake_schema:
  - name: customer_id
  - name: transaction_time
  - name: product_id
  - name: quantity_sold

# Example Caching Configuration
cache_config:
  backend: redis
  host: localhost
  port: 6379
```

**Flow:**

1. External Data Source
2. Data Lake Storage
3. ETL Pipeline Processing
4. Cache Mechanism -Redis/Memcached
5. Real-Time Processing
6. Model Consumption

