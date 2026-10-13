---
type: "practice"
difficulty: "Deep"
---

# Argue whether the warehouse should ever be system-of-record for any feature, or purely a derived analytical copy. What goes wrong for ML when teams start treating a derived feature table as authoritative for operational decisions?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Warehouse as System of Record](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Warehouse_as_System_of_Record.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a robust data architecture, the warehouse typically serves as a derived analytical copy rather than a system-of-record (SOR) for any feature. This distinction is critical because treating a derived feature table as an SOR can lead to inconsistencies, operational issues, and suboptimal decision-making in ML models. The key issue arises when teams rely on derived features for real-time or near-real-time decisions; such reliance introduces risks that can undermine the integrity of both the data and the ML models.

The core idea is this: In a well-designed architecture, the warehouse should only contain immutable data processed from SORs, ensuring consistency and traceability. Any operational system requiring current state data should directly access the SORs or use a near real-time data mesh to update its local cache.

#### Triaging the Question

The question asks us to argue whether the warehouse should ever be used as an SOR for any feature, versus it being purely a derived analytical copy. The interviewer is probing our understanding of data lifecycle management, consistency strategies, and the implications of treating derived features as authoritative in operational contexts. Key assumptions include:

Data Freshness: Real-time or near-real-time decisions require current state data.
Data Integrity: SORs are considered authoritative for historical and transactional operations.

#### First-Principles Solution
1. Data Architecture Overview

A modern data architecture often includes three key components: the source of truth (SOR), a data lake, and an analytics warehouse. The SOR contains real-time or near-real-time data that is immutable once committed. The data lake stores raw and semi-processed data for long-term storage and analysis. The warehouse processes and aggregates data from the lake to create derived features used for analytics.

2. Why the Warehouse Should Not Be an SOR

#### Cons
istency Issues: Derived features in the warehouse are often calculated based on historical or aggregated data, which may not reflect current state information.
Operational Impact: Relying on a derived feature table can lead to inconsistent decisions if the underlying data is outdated or incorrect.
Latency and Scalability: Real-time operational systems require immediate updates; deriving features in batch mode does not meet these requirements.
3. Practical Examples

#### Cons
ider an e-commerce platform where customer purchase history influences product recommendations. If the warehouse were used as an SOR for this feature, changes to a customer’s recent purchases would be propagated through the pipeline and stored there before being reflected in the recommendation system. This delay could result in stale recommendations.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, treating the warehouse as an SOR can lead to significant operational issues and suboptimal ML model performance. It’s crucial for teams to understand that the warehouse should serve purely derived analytical purposes, while operational systems need direct access to the source of truth.

#### Common mistakes
include:

Treating Derived Features as Authoritative: Teams sometimes view derived features in the warehouse as authoritative for real-time decisions.
Ignoring Data Freshness: Not ensuring data is up-to-date can lead to incorrect conclusions and suboptimal decisions.

#### Follow-Ups
How Would You Design a Real-Time Pipeline?
What Are the Trade-offs Between Using SORs Directly vs. Derived Features in Warehouses?
Can You Provide an Example of When It Might Be Justified to Use Warehouse Data as SOR for Certain Operations?

**Comparison:**

```
Step	Description
1	Read current customer activity logs from the source of truth.
2	Process and aggregate data to update derived features in the warehouse.
3	Ensure the data is immutable once written to the warehouse.
```

**Code:**

```python
# Example of a simple ETL job using Airflow for data from SOR -> Lake -> Warehouse
from airflow import DAG
from airflow.providers.apache.spark.operators.spark_submit import SparkSubmitOperator

dag = DAG(
    'data_pipeline',
    description='Data pipeline from source of truth to warehouse',
    schedule_interval='@daily',
)

etl_job = SparkSubmitOperator(
    task_id='transform_and_load_data',
    application='/path/to/transform.py',  # Script that reads SOR, processes and writes to Warehouse
    dag=dag,
)
```

**Example input / output:**

```text
Input:
- Real-time customer purchase events from SalesDatabase.

Output:
- Derived CustomerPurchases feature in AnalyticalWarehouse.
```

**Data:**

```json
{
  "source_of_truth": {
    "name": "SalesDatabase",
    "tables": ["Customers", "Orders"]
  },
  "data_lake": {
    "name": "RawDataLake",
    "staging_tables": ["CustomerActivityLogs", "OrderDetails"]
  },
  "warehouse": {
    "name": "AnalyticalWarehouse",
    "derived_features": ["CustomerPurchases", "ProductPopularityMetrics"]
  }
}
```

**Flow:**

1. Source of Truth (SOR)
2. Lake
3. Warehouse (Derived Features)
4. Analytics Systems

