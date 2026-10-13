---
type: "practice"
difficulty: "Deep"
---

# How do you design a bronze-silver-gold medallion architecture and what specific responsibilities and quality guarantees does each layer carry?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ GDPR Deletion in Feature Pipelines](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/GDPR_Deletion_in_Feature_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a bronze-silver-gold (B-S-G) medallion architecture, we need to think about how to capture raw data, clean it, transform it, and store it in different stages of granularity and maturity. The bronze layer is for raw, unprocessed data, the silver layer for cleaned and validated data, and the gold layer for refined, aggregated insights that are ready for business use.

The key components we need to consider include data ingestion, transformation logic, validation steps, and storage mechanisms. Each layer should have specific responsibilities and quality guarantees:

Bronze Layer: Ingest raw data as it arrives.
Silver Layer: Perform initial cleaning and validation of the bronze-layer data before moving on.
Gold Layer: Aggregate and refine silver-layer data to provide business insights.

Let's break this down step-by-step from first principles:

#### Triage
Understanding the Question

The question is asking about designing a B-S-G architecture, which involves understanding how each layer processes raw data. The goal is to ensure that data flows smoothly through these layers while maintaining quality and integrity at every stage. We need to consider what happens in each layer, the tools we might use, and the best practices for ensuring high-quality data.

#### Assumptions
We are using an ETL tool like Apache Spark or Airflow.
The raw data can come from various sources (databases, APIs, files).
We need to define clear quality control checks at each layer.
Different storage mechanisms might be used for different layers.
Step-by-Step Solution
Bronze Layer

The bronze layer is responsible for capturing the raw, incoming data. This could include reading data from a variety of sources such as databases (e.g., MySQL), file systems (e.g., CSV, JSON), or streaming platforms (e.g., Kafka).

Responsibilities:

Ingest raw data.
Store it in a temporary location.

Quality Guarantees:

Ensure that all incoming data is captured and stored reliably.
Minimal transformation—basically, storing as-is for now.
Silver Layer

The silver layer takes the raw data from the bronze layer and performs initial cleaning and validation. This stage is crucial for ensuring data quality before moving to more complex transformations in the gold layer.

Responsibilities:

Cleanse the data by handling missing values, removing duplicates, etc.
Validate the data against predefined rules (e.g., date formats, value ranges).
Store processed data in a structured format.

Quality Guarantees:

Data should be clean and consistent before moving to the gold layer.
Quality checks are performed to ensure no invalid or inconsistent data passes through.
Gold Layer

The gold layer is where we aggregate and refine the silver-layer data to provide meaningful insights. This could involve complex queries, machine learning models, and advanced analytics.

Responsibilities:

Aggregate data by various dimensions (e.g., time, region).
Perform complex transformations and calculations.
Store refined data in a format that supports quick querying and analysis.

Quality Guarantees:

Data should be aggregated to provide meaningful insights for business use.
Ensure that the gold-layer data is accurate and relevant.

#### Implementation
Using Airflow

We can implement this architecture using Apache Airflow, which provides a workflow orchestration system. Here’s how we might structure our tasks:

AIRFLOW
from airflow import DAG
from airflow.providers.apache.spark.operators.spark_submit import SparkSubmitOperator

default_args = {
    'owner': 'data_engineer',
    'start_date': datetime(2023, 10, 1),
}

with DAG('bronze_silver_gold_dag', default_args=default_args, schedule_interval='@daily') as dag:

    # Bronze Layer
    bronze_task = SparkSubmitOperator(
        task_id='bronze_layer',
        application='/path/to/bronze_script.py',
        conn_id='spark_default'
    )

    # Silver Layer
    silver_task = SparkSubmitOperator(
        task_id='silver_layer',
        application='/path/to/silver_script.py',
        conn_id='spark_default'
    )

    # Gold Layer
    gold_task = SparkSubmitOperator(
        task_id='gold_layer',
        application='/path/to/gold_script.py',
        conn_id='spark_default'
    )

bronze_task >> silver_task >> gold_task

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the B-S-G architecture ensures that raw data is captured reliably, cleaned appropriately, and transformed into useful insights. Each layer has specific responsibilities and quality guarantees to ensure data integrity.

#### Common mistakes
include:

Overlooking initial cleaning in the silver layer.
Failing to validate data thoroughly.
Not performing sufficient aggregation and refinement in the gold layer.

For follow-ups, we might discuss tools like Apache Beam for more robust data processing, or explore how to integrate machine learning models directly into these layers for predictive analytics.

**Code:**

```python
from airflow import DAG
from airflow.providers.apache.spark.operators.spark_submit import SparkSubmitOperator

default_args = {
    'owner': 'data_engineer',
    'start_date': datetime(2023, 10, 1),
}

with DAG('bronze_silver_gold_dag', default_args=default_args, schedule_interval='@daily') as dag:

    # Bronze Layer
    bronze_task = SparkSubmitOperator(
        task_id='bronze_layer',
        application='/path/to/bronze_script.py',
        conn_id='spark_default'
    )

    # Silver Layer
    silver_task = SparkSubmitOperator(
        task_id='silver_layer',
        application='/path/to/silver_script.py',
        conn_id='spark_default'
    )

    # Gold Layer
    gold_task = SparkSubmitOperator(
        task_id='gold_layer',
        application='/path/to/gold_script.py',
        conn_id='spark_default'
    )

bronze_task >> silver_task >> gold_task
```

**Flow:**

1. Yes
2. No
3. Bronze Layer
4. Is Data Clean?
5. Silver Layer
6. Error Handling
7. Gold Layer

