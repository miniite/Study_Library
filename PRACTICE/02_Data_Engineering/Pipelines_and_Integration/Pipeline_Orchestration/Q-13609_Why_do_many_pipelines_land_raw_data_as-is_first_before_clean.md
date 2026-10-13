---
type: "practice"
difficulty: "Intermediate"
---

# Why do many pipelines land raw data as-is first before cleaning it, instead of cleaning during extraction?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ GDPR Deletion in Feature Pipelines](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/GDPR_Deletion_in_Feature_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a data pipeline, landing raw data "as is" and then applying transformations or cleaning steps later can be more advantageous than cleaning data at the source. This approach addresses several practical concerns related to performance, storage efficiency, and fault isolation.

Firstly, performance. Data extraction involves significant computational resources, especially for large datasets. If we apply complex cleaning operations during extraction, it could slow down the entire pipeline process. By landing raw data first, we can leverage more efficient ETL (Extract, Transform, Load) tools that are optimized for bulk data handling and storage.

Secondly, storage efficiency. Raw data often takes up less space than cleaned data because not all transformations may be necessary or beneficial at this stage. Cleaning operations might introduce redundant columns, increase data size, or even remove valuable information temporarily. Storing raw data allows us to optimize the data footprint before applying heavy cleaning.

Thirdly, fault isolation. Pipelines can fail due to various reasons, such as network issues, bugs in transformation logic, or unexpected data patterns. By separating the extraction and cleaning phases, we isolate potential failures. If a cleaning step fails, it doesn't impact the integrity of the raw data, allowing for easier debugging and recovery.

Lastly, flexibility. Landing raw data gives us more flexibility to experiment with different cleaning strategies without impacting the core data flow. This is particularly useful in agile development environments where requirements can change frequently.

To illustrate these points further:

#### Example Scenario
Imagine a financial transaction system that logs every transaction in real-time. The extraction process captures this raw data and writes it directly to storage. Later, during processing, we apply cleaning steps such as removing duplicate entries, correcting timestamps, or filtering out irrelevant transactions.

#### Pros
of landing raw data:
Easier debugging due to separate stages.
Enhanced performance by optimizing bulk operations.
Greater flexibility in handling various transformations.

In summary, the approach of landing raw data first and applying cleaning later is a common practice because it offers benefits in terms of performance optimization, storage efficiency, fault isolation, and flexibility. This method ensures that the pipeline remains robust and adaptable to changing requirements.

**Code:**

```python
# Example code for extracting and storing raw data using Apache Spark
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("RawDataPipeline").getOrCreate()

raw_data_df = spark.read.format("csv").option("header", "true").load("/path/to/transactions")
raw_data_df.write.parquet("/data/raw_transactions")
```

**Example input / output:**

```
Input: Real-time transaction logs written to a CSV file.
Output: Raw transaction data stored in Parquet format.
SHELL
Copy
/path/to/transactions

Converted to Parquet:

SHELL
Copy
/data/raw_transactions
```

**Data:**

```json
{
  "transaction_id": int,
  "timestamp": string,
  "amount": float,
  "merchant_id": int,
  "category": string,
  "user_id": int
}
```

