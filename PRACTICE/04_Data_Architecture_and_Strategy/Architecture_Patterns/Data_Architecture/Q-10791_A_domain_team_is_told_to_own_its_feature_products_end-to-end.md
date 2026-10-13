---
type: "practice"
difficulty: "Deep"
---

# A domain team is told to own its feature products end-to-end but has no data-engineering skill. How do you resolve the tension between decentralized ML data ownership and uneven team maturity? Define exactly what the paved road provides for free.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cross-domain Joins in a Mesh](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Cross-domain_Joins_in_a_Mesh.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the tension between decentralized ML data ownership and uneven team maturity, we need to strike a balance by providing a standardized framework that ensures consistency in data quality and access while allowing domain teams autonomy. The "paved road" concept here refers to a set of established practices, tools, and guidelines that minimize friction for the domain teams, enabling them to focus on their core competencies without reinventing wheels.

In this scenario, the key challenge is to design a scalable, efficient data architecture that caters to the varying levels of technical maturity within the team. Let's break down the solution from first principles:

Understand the Scope: The domain team needs to own its feature products end-to-end but lacks necessary skills in data engineering. This means they need robust data pipelines, data storage, and governance mechanisms without having to build these from scratch.

Identify Core Requirements: Define what aspects of data management can be standardized across all teams, such as data schemas, ETL processes, and security policies. These are the "paved roads" that provide free services, allowing the team to focus on their specific product features without getting bogged down by infrastructure details.

Data Lake vs Data Warehouse: For a heterogeneous team with varying levels of maturity, it’s beneficial to leverage a combination of data lakes and data warehouses. A data lake can handle diverse raw data sources and complex transformations, while a data warehouse serves as an aggregated, clean dataset for reporting and analysis. This hybrid approach ensures that the domain teams have access to both raw and processed data.

Automated Data Pipelines: Implement reusable ETL pipelines using platforms like Apache Airflow or AWS Glue. These tools automate data extraction, transformation, and loading processes, reducing the overhead on the domain team. The paved roads in this context include pre-built connectors for common data sources and streamlined workflow management.

Data Governance Standards: Establish a set of governance standards that define data access controls, data quality checks, and lineage tracking. Tools like Apache Atlas or AWS Glue Data Catalog can help automate these processes, ensuring compliance and transparency without requiring deep expertise from the domain team.

Documentation and Training: Provide comprehensive documentation and training materials to help the domain teams understand how to use the provided infrastructure effectively. This includes tutorials on data cleaning, feature engineering, and model validation best practices.

Support and Maintenance: Ensure that a dedicated support team is available to assist with any issues or questions the domain team might have. Regular maintenance updates and bug fixes should be handled by this support team to minimize disruptions for the end-users.

In summary, the "paved road" in this context includes predefined data storage solutions (data lake/data warehouse), automated ETL pipelines, standardized governance policies, and comprehensive documentation/support mechanisms. These elements ensure that even teams with varying levels of maturity can effectively manage their ML projects without needing to build everything from scratch.

#### Common mistakes
include over-engineering the solution or underestimating the need for domain-specific expertise. Always aim for a balance between standardization and flexibility to cater to different team needs.

**Comparison:**

```
Task	Description
ETL	Extract, Transform, Load
Data Lake	Raw and unprocessed data storage
Data Warehouse	Aggregated, cleaned data for BI
```

**Code:**

```python
# Example ETL Pipeline using Apache Airflow
from airflow import DAG
from airflow.providers.apache.spark.operators.spark_submit import SparkSubmitOperator

default_args = {
    'owner': 'data_engineer',
    'retries': 1,
}

dag = DAG(
    'example_etl_dag',
    default_args=default_args,
    schedule_interval='@daily',
)

etl_task = SparkSubmitOperator(
    task_id='run_etl_job',
    application='/path/to/etl_script.py',  # Path to ETL script
    conn_id='spark_default',
    dag=dag,
)
```

**Example input / output:**

```

Input Data:

JSON
Copy
[
  {"id": 1, "timestamp": "2023-10-01", "feature1": 0.5, "label": true},
  {"id": 2, "timestamp": "2023-10-02", "feature1": 0.7, "label": false}
]

Output Data:

JSON
Copy
[
  {"id": 1, "transformed_feature": 0.45},
  {"id": 2, "transformed_feature": 0.63}
]
```

**Data:**

```json
{
  "schema": {
    "id": "int",
    "timestamp": "datetime",
    "feature1": "float",
    "label": "bool"
  }
}
```

**Flow:**

1. Data Ingestion
2. Data Lake
3. ETL Pipelines
4. Data Warehouse
5. BI/ML Models

