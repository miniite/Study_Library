---
type: "practice"
difficulty: "Deep"
---

# Your warehouse queries for training hit a concurrency limit during a big experiment sweep. How do you isolate ML workloads from analytics workloads?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Rollups vs On-demand Queries](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Rollups_vs_On-demand_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of concurrent queries hitting limits in your data warehouse, we need to design a robust and scalable data architecture that effectively isolates machine learning (ML) workloads from analytical ones. This involves understanding the underlying concurrency constraints, assessing the current architecture, and proposing a strategic redesign.

#### Triage

The question is essentially about designing a data architecture that can handle concurrent queries without overloading your data warehouse. The interviewer might be probing into how well you understand the trade-offs between various architectural choices, such as data lakes versus warehouses, and how to manage concurrency in ML workloads. Key assumptions include the current limitations of the data warehouse, the volume of concurrent queries during experiments, and the importance of maintaining performance for both analytics and ML tasks.

#### First-Principles Solution

To isolate ML workloads from analytical ones, we need to consider a multi-tiered architecture that leverages different storage solutions. The primary components are:

Data Lake: Stores raw data in its original format.
Data Warehouse: Aggregates and cleans the data for analytics.
ML Workload Storage: Specialized storage or managed services tailored for ML models.
Step-by-Step Solution

Identify Concurrency Issues:

Understand the specific concurrency limits that are being hit, whether it’s related to read/write operations, memory usage, or compute resources.
Gather metrics on query performance and resource utilization during experiment sweeps.

Assess Current Architecture:

Evaluate if a traditional data warehouse is causing bottlenecks due to its shared nature for both analytics and ML tasks.

#### Cons
ider the data lake architecture that can store raw data without immediate processing, allowing parallel queries from different workloads.

Propose a Multi-Tiered Architecture:

Data Lake: Store raw data in a scalable and cost-effective solution like Amazon S3 or Google Cloud Storage. This tier allows for flexible querying and does not impose constraints on ML operations.
MARKDOWN
[Data Lake]
  +-- Raw Data (e.g., CSV, Parquet)
  +-- Versioned Data
Data Warehouse: Aggregates and cleans data from the data lake into a schema optimized for analytics. This tier should be managed to ensure it can handle high concurrency during regular operations.
MARKDOWN
[Data Warehouse]
  +-- Aggregated Tables
  +-- Cleaned Data
ML Workload Storage: Use managed ML services like AWS SageMaker or Azure Machine Learning Studio, which provide dedicated storage and compute resources. These services can also scale out automatically during big experiment sweeps.
MARKDOWN
[ML Workload Storage]
  +-- Model Artifacts
  +-- Training Data (from Data Lake)

Implement Isolation:

Use access controls and network segmentation to ensure that ML workloads do not interfere with analytical queries.
Implement query throttling or rate limiting in the data warehouse to manage concurrent query loads during experiment sweeps.

Monitor and Optimize:

Continuously monitor resource usage and performance metrics across all tiers.
Optimize schema, indexing, and storage configurations based on query patterns and workload characteristics.

#### Closing Summary

By designing a multi-tiered architecture that leverages data lakes for raw storage, traditional warehouses for analytics, and managed ML services for training, we can effectively isolate different workloads and ensure they do not overload each other. This approach provides flexibility, scalability, and improved performance by managing concurrency at multiple levels.

#### Common Mistakes/Follow-Ups
Not Considering Data Freshness: Ensuring that the data lake is updated frequently to reflect the latest training data.
Overlooking Network Segmentation: Failing to properly segment networks to prevent ML workloads from overwhelming analytics queries.
Ignoring Query Optimization: Not optimizing queries in the warehouse for high concurrency, leading to performance degradation.

**Comparison:**

```
Tier	Storage Solution	Use Case
Data Lake	Amazon S3, Google Cloud Storage	Raw data storage for flexibility and scalability
Data Warehouse	AWS Redshift, Google BigQuery	Aggregated and cleaned data for analytics queries
ML Workload Storage	AWS SageMaker, Azure Machine Learning Studio	Specialized storage and compute resources for model training
```

**Code:**

```python
# Example of a simple query throttling mechanism
def throttle_queries(query, threshold=10):
    current_load = get_current_query_load()  # Assume this function exists
    if current_load >= threshold:
        raise Exception("Query load exceeded; please try later.")
    else:
        execute_query(query)
```

**Example input / output:**

```text
Input:
- Experiment sweep initiated with high concurrency

Output:
- Analytical queries continue to run smoothly
- ML workloads are isolated and do not interfere
- Performance metrics show improved stability during experiments
```

**Data:**

```json
{
  "DataLake": {
    "RawData": [
      {"file_name": "raw_data_1.csv", "size": 500MB},
      {"file_name": "raw_data_2.parquet", "size": 800MB}
    ],
    "VersionedData": [
      {"version": 1, "timestamp": "2023-10-01"},
      {"version": 2, "timestamp": "2023-10-05"}
    ]
  },
  "DataWarehouse": {
    "AggregatedTables": [
      {"table_name": "customer_metrics", "schema_version": 2}
    ],
    "CleanedData": [
      {"file_name": "clean_data_1.csv", "size": 300MB},
      {"file_name": "clean_data_2.parquet", "size": 450MB}
    ]
  },
  "MLWorkloadStorage": {
    "ModelArtifacts": [
      {"model_name": "v1", "version": 1, "timestamp": "2023-10-05"},
      {"model_name": "v2", "version": 2, "timestamp": "2023-10-10"}
    ],
    "TrainingData": [
      {"file_name": "training_data_1.csv", "size": 600MB},
      {"file_name": "training_data_2.parquet", "size": 900MB}
    ]
  }
}
```

**Flow:**

1. Data Lake
2. Store Raw Data
3. Version Control
4. Data Warehouse
5. Aggregate & Clean
6. ML Workload Storage
7. Maintain Model Artifacts

