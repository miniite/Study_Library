---
type: "practice"
difficulty: "Basic"
---

# What are the medallion architecture layers (bronze silver gold) and what does each hold?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Lake Catalogs](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Lake_Catalogs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The medallion architecture is a layered approach used in data warehousing, particularly within data lakes and lakehouses, to organize raw data into different tiers based on its processed state and usage patterns. It helps in maintaining a balance between storage costs, query performance, and data freshness.

TL;DR
Bronze Layer: Raw, unprocessed data straight from sources.
Silver Layer: Cleaned and validated data ready for analytics.
Gold Layer: Aggregated and summarized data optimized for reporting and BI tools.

#### Triage
What's Being Asked & Assumptions

The question is asking to explain the three key layers of medallion architecture in a data lake or lakehouse. This involves understanding how raw data is transformed into usable analytics-ready datasets, considering storage costs and processing efficiency.

#### First-Principles Solution

Bronze Layer:

Purpose: Store unprocessed raw data directly from sources like databases, IoT devices, or other systems.
Data Types: Includes logs, transaction records, sensor data, etc., in their original format.
Storage Considerations: Uses cost-effective storage solutions to accommodate large volumes of raw data. Often stored in formats like Parquet, ORC, JSON, or Avro for efficient querying.

Silver Layer:

Purpose: Clean and validate the data by applying transformations and cleansing rules.
Data Types: Data is now consistent and ready for analysis but not yet fully optimized.
Storage Considerations: Utilizes more performant storage solutions that balance cost and performance, such as S3 or HDFS. The goal here is to ensure the data is reliable and can be efficiently queried.

Gold Layer:

Purpose: Aggregate and summarize data for business reporting and analytics.
Data Types: Summarized metrics like monthly sales figures, customer churn rates, etc., which are optimized for fast queries by analysts and BI tools.
Storage Considerations: Stores in highly performant databases or data warehousing solutions like Redshift or BigQuery to enable quick access and reporting.

#### Closing Summary

The medallion architecture is a strategic approach that organizes data through three layers: Bronze (raw), Silver (cleaned/transformed), and Gold (analyzed). Each layer serves a distinct purpose, optimizing the balance between storage costs, data freshness, and analytical performance.

#### Common Mistakes/Follow-Ups
Common Mistake: Forgetting to mention that the architecture is designed for cost optimization by storing raw data cheaply and processing it as needed.

#### Follow-Up
Discuss how different layers can be managed differently in terms of storage lifecycle policies and backup strategies. Also, consider the impact on query performance across these layers.

**Comparison:**

| Layer | Purpose                       | Data Types                    | Storage Format  | Cost/Efficiency     |
|-------|-------------------------------|-------------------------------|------------------|--------------------|
| Bronze | Raw, unprocessed data         | Logs, transactions, sensor    | Parquet          | Cost-efficient     |
| Silver | Cleaned and validated data   | Consistent, ready for analysis| Parquet          | Balanced cost/perf  |
| Gold   | Aggregated & summarized data | Metrics like sales           | Redshift         | Performance-optimized|

**Code:**

```python
# Example code to read bronze layer data from S3 and write to silver layer
import pandas as pd

def process_bronze_to_silver(source_path, destination_path):
    # Read raw data
    raw_data = pd.read_csv(source_path)

    # Apply cleaning rules (example: drop duplicates, fill nulls, etc.)
    cleaned_data = clean_raw_data(raw_data)

    # Write processed data to silver layer storage
    cleaned_data.to_parquet(destination_path)

def clean_raw_data(df):
    # Example cleaning logic
    df.drop_duplicates(inplace=True)
    df.fillna(value=0, inplace=True)  # Fill null values with zero as an example

    return df
```

**Example input / output:**

```text
# Example of data movement from Bronze to Silver layer

Input (Bronze Layer):
Transaction Logs:
- TransactionID, UserID, ItemID, Timestamp, Amount

Output (Silver Layer):
Cleaned and Processed Data:
- UserID, ItemID, TotalAmount, Date

Query Performance in Gold Layer:
SELECT SUM(TotalAmount) FROM gold_layer WHERE Date BETWEEN '2023-01-01' AND '2023-12-31';
```

**Data:**

```json
{
  "bronze_layer": {
    "source": ["db_transactions", "sensor_logs"],
    "storage_format": "parquet",
    "cost_efficient_storage": true
  },
  "silver_layer": {
    "source": ["bronze_layer"],
    "cleaning_rules": [
      { "operation": "drop_duplicates" },
      { "operation": "fill_nulls", "value": 0 }
    ],
    "storage_format": "parquet",
    "cost_efficient_storage": false
  },
  "gold_layer": {
    "source": ["silver_layer"],
    "aggregation_rules": [
      { "metric": "monthly_sales", "aggregation": "SUM" },
      { "metric": "customer_churn_rate", "aggregation": "COUNT" }
    ],
    "storage_format": "redshift",
    "performance_optimized_storage": true
  }
}
```

**Flow:**

1. Raw Data
2. Bronze Layer
3. Silver Layer
4. Gold Layer

