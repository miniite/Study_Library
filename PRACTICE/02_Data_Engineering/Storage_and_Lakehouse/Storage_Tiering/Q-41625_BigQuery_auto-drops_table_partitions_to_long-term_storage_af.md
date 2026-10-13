---
type: "practice"
difficulty: "Deep"
---

# BigQuery auto-drops table partitions to long-term storage after 90 days of no edits. How do you exploit or work around that behavior in a partitioning + tiering design?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Storage Tiering Design](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Storage_Tiering/Storage_Tiering_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we are discussing how to leverage BigQuery's auto-tiering and partitioning features while working around its automatic dropping of table partitions after 90 days of inactivity. The goal is to design a storage tiering strategy that maximizes cost efficiency without losing data access or compromising on query performance.

First, let’s break down the problem:

Behavior Understanding: BigQuery automatically moves old partitioned tables to cheaper storage tiers (e.g., Coldline) after 90 days of no writes.
Key Challenges: Ensuring data availability and maintaining efficient query performance while managing costs.

#### Assumptions
We assume a need for long-term retention of historical data but also wish to minimize storage costs.

To address this, we can employ the following strategies:

Partitioning Strategy: Design partitions based on business needs rather than purely date-based.
Data Archiving: Implement a custom archiving mechanism using BigQuery tables and external storage.
Managed Storage Classes: Use BigQuery's managed storage classes to balance cost and performance.
Step-by-Step Solution

Partitioning Strategy:

Business-Based Partitions: Instead of auto-partitioning by date, consider partitioning based on business relevance. For example, if you have a transaction table with customer IDs, partition it by customer segment or region.
SQL
CREATE TABLE sales (
  order_date DATE,
  customer_id INT64,
  product_id INT64,
  amount FLOAT64
) PARTITION BY RANGE(customer_segment)
Regular Maintenance: Manually run maintenance tasks to optimize the partitioning and move data as needed.

Data Archiving:

External Storage: Use BigQuery's support for external tables pointing to Google Cloud Storage (GCS) buckets.
SQL
CREATE TABLE archive (
  order_date DATE,
  customer_id INT64,
  product_id INT64,
  amount FLOAT64
) OPTIONS (
  location='US',
  require_partition_filter=true
);
Custom Scripting: Develop a script to periodically copy relevant data from BigQuery to GCS and vice versa, ensuring that you retain necessary historical data.

Managed Storage Classes:

Coldline Storage: Use Coldline storage for infrequently accessed but still critical data.
Economic Analysis: Regularly review the costs associated with different storage tiers and adjust as needed to optimize cost-to-benefit ratio.

#### Closing Summary

By employing a strategic partitioning approach, utilizing external storage for archiving, and leveraging BigQuery's managed storage classes, we can effectively manage long-term data retention while minimizing costs. This design ensures that critical data remains accessible and query performance is maintained.

#### Common Mistakes/Follow-Ups

Over-reliance on Auto-Tiering: Not fully understanding the limitations of auto-tiering can lead to unexpected data loss.
Inadequate Data Archiving: Failing to implement a robust archiving strategy may result in incomplete historical data retention.
Cost Management: Ignoring cost management and not periodically reviewing storage classes can lead to increased expenses.

**Comparison:**

| Partitioning Strategy | Action |
|-----------------------|--------|
| Customer Segment      | Manual maintenance and archiving to GCS. |
| Date-based            | Auto-tiering by BigQuery.                 |

**Code:**

```

-- Example of creating a partitioned table with business-based partitions CREATE TABLE sales ( order_date DATE, customer_id INT64, product_id INT64, amount FLOAT64 ) PARTITION BY RANGE(customer_segment);

CODE
Copy

-- Script for copying data to GCS and back
```python
import subprocess

def copy_to_gcs():
    cmd = "bq extract --location=US sales:archive:2021 order_data_*.csv gs://my_bucket/archive/"
    subprocess.run(cmd, shell=True)

def copy_from_gcs():
    cmd = "bq load --source_format=CSV sales:archive order_data_*.csv gs://my_bucket/archive/*.csv"
    subprocess.run(cmd, shell=True)
```

**Example input / output:**

```

Example Input->Output:

Input: A BigQuery table with historical sales data partitioned by customer segment.
Output: Periodic archiving to GCS and manual maintenance of partitions.
SQL
Copy
-- Example SQL for manual maintenance
ALTER TABLE sales RENAME TO sales_archived;
CREATE TABLE sales (
  order_date DATE,
  customer_id INT64,
  product_id INT64,
  amount FLOAT64
) PARTITION BY RANGE(customer_segment);
```

**Data:**

```json
{
  "storageClass": "COLDLINE",
  "location": "US",
  "requirePartitionFilter": true
}
```

**Flow:**

1. BigQuery Table
2. Manual Partition Maintenance
3. Create New Table withUpdated Partitions
4. Audit and Archive Data
5. Periodic Archiving to GCS
6. Auto-Tiering by BigQuery
7. Tier Data to Coldline

