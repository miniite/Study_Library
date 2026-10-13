---
type: "practice"
difficulty: "Expert"
---

# A regulator requires you to reproduce the exact training dataset for any credit model on demand for 7 years. Architect the immutable snapshotting and retention layer across a lakehouse without unbounded storage growth.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Warehouse Vendor Exit Strategy](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Warehouse_Vendor_Exit_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect an immutable snapshotting and retention layer in a lakehouse that allows reproducing any training dataset for credit models on demand over a 7-year period, we need to focus on a robust data architecture design. This involves understanding the regulatory requirements, designing a scalable storage mechanism, and implementing a reliable data management strategy.

#### Triaging the Question

The core of this question is about ensuring immutable snapshots (data integrity) while managing storage growth effectively without violating the regulator's mandate for 7 years of data retention. The key aspects to consider are:

Data Integrity: Ensuring that once data is ingested, it cannot be altered.
Retrieval Requirements: Being able to reproduce datasets exactly as they were at any point in time.
Storage Management: Managing storage growth without unbounded increases.

#### First-Principles Solution

Immutable Snapshots:

Use a version-controlled file system or a distributed storage system that supports immutable writes (e.g., HDFS with append-only semantics).
Implement checksums and metadata to verify data integrity at any point in time.

Data Versioning:

Employ a timestamp-based naming convention for files, where each ingestion is versioned uniquely (e.g., credit_model_training_data-YYYY-MM-DD-HH-mm-ss).

Storage Layout:

Organize data by date ranges or model training sessions to facilitate efficient retrieval and manage storage growth.
Use a mesh architecture that allows for multiple access patterns while maintaining data integrity.

Metadata Management:

Store metadata about each version, including dependencies on other versions, lineage information, and relevant context (e.g., source systems, transformations applied).

Data Retention Policy:

Implement a policy to automatically delete old snapshots after 7 years unless explicitly retained for compliance reasons.
Ensure that the retention period is enforced by the storage system itself or through a lifecycle management service.
Detailed Solution

Storage System Selection:

Choose a distributed file system like HDFS (Hadoop Distributed File System) with append-only semantics to ensure data immutability.
Alternatively, use an object storage system like Amazon S3 with versioning enabled and lifecycle policies configured for automatic deletion after 7 years.

Data Ingestion:

Implement a data ingestion pipeline that ingests raw data into the lakehouse and writes it as immutable snapshots.
Use Apache Airflow or similar orchestration tools to automate the ingestion process, ensuring consistent naming conventions and metadata tagging.

Metadata Management:

Store metadata in a relational database like PostgreSQL or a NoSQL store like Cassandra.
The metadata should include details about each snapshot (timestamp, version number, source system, transformations applied), lineage information, and any relevant annotations.

Data Retrieval Mechanism:

Develop an API that allows users to request specific versions of the training dataset by timestamp or model session identifier.
Use a query engine like Apache Presto or SQL over S3 to retrieve data from versioned storage efficiently.

Storage Optimization:

Implement data compression and deduplication strategies to optimize storage usage.
Employ a mesh architecture where multiple access patterns can coexist without impacting the immutability of individual snapshots.

#### Closing Summary

To meet the regulator's requirements, we need an immutable snapshotting and retention layer that ensures data integrity and allows for exact reproducibility. This involves using version-controlled file systems or object storage with lifecycle policies, implementing robust metadata management, and developing a retrieval mechanism that can handle requests efficiently without unbounded storage growth.

#### Common Mistakes/Follow-Ups
Failure to Implement Data Integrity Checks: Failing to verify the integrity of data snapshots could lead to unrecoverable issues when reproducing datasets.
Lack of Metadata Management: Poor metadata management can make it difficult to track and retrieve specific versions of training data, leading to inefficiencies or errors in model training.
Inadequate Storage Optimization: Failing to optimize storage usage through compression, deduplication, and lifecycle management policies can result in excessive costs or performance bottlenecks.

**Comparison:**

| Column Name | Data Type |
|-------------|-----------|
| id          | int       |
| timestamp   | string    |
| version     | string    |
| source_system | string   |
| transformations_applied | array<string> |
| lineage     | object    |
| annotations | object    |

**Code:**

```python
# Example Airflow DAG for data ingestion into a version-controlled file system
from airflow import DAG
from datetime import datetime
import os

default_args = {
    'owner': 'data_engineer',
    'depends_on_past': False,
    'start_date': datetime(2023, 1, 1),
}

dag = DAG(
    'credit_model_data_ingestion',
    default_args=default_args,
    schedule_interval='@daily',
)

def ingest_data(**kwargs):
    source_path = '/path/to/source/data'
    target_path = f'/path/to/target/snapshots/{datetime.now().strftime("%Y-%m-%d-%H-%M-%S")}'

    # Perform data ingestion and write as immutable snapshot
    os.makedirs(target_path, exist_ok=True)
    shutil.copyfile(source_path, os.path.join(target_path, 'data.csv'))

ingest_task = PythonOperator(
    task_id='ingest_data',
    python_callable=ingest_data,
    dag=dag,
)
```

**Example input / output:**

```text
# Sample metadata input and output for data retrieval
metadata_input = {
  "id": 12345,
  "timestamp": "2023-06-15T08:00:00Z",
  "version": "v1.0",
  "source_system": "bank_transaction",
  "transformations_applied": ["filter_outliers", "normalize_data"],
  "lineage": {
    "parents": [12344, 12346],
    "children": []
  },
  "annotations": {
    "model_version": "v0.9"
  }
}

metadata_output = {
  "id": 12345,
  "timestamp": "2023-06-15T08:00:00Z",
  "version": "v1.0",
  "source_system": "bank_transaction",
  "transformations_applied": ["filter_outliers", "normalize_data"],
  "lineage": {
    "parents": [12344, 12346],
    "children": []
  },
  "annotations": {
    "model_version": "v0.9"
  }
}
```

**Data:**

```json
{
  "schema": {
    "id": int,
    "timestamp": string,
    "version": string,
    "source_system": string,
    "transformations_applied": array<string>,
    "lineage": object,
    "annotations": object
  }
}
```

**Flow:**

1. Yes
2. No
3. Data Ingestion
4. Is Data Versioned?
5. Write to Immutable Storage
6. Log Error and Retry Later
7. Update Metadata in RelationalDB/NOSQL Store
8. Store Metadata for Retrieval
9. Data Retention Policy
10. Lifecycle Management DeletesOld Snapshots After 7 YearsUnless Explicitly Retained

