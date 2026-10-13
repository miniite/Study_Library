---
type: "practice"
difficulty: "Deep"
---

# Your Spark job that reads a partitioned table works on a sample in dev but times out on the full table in prod. What assumptions about data layout or skew likely broke between the two?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ End to End No Data Loss](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/End_to_End_No_Data_Loss.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a scenario where your Spark job behaves differently between development and production environments, especially when transitioning from a small sample to a large dataset, one of the key issues often lies in data skew or layout differences. Here’s how we can systematically approach this problem:

TL;DR: The core issue is likely related to data skew and partitioning differences between dev and prod environments. By understanding the data distribution and partition strategy, you can optimize Spark jobs for better performance.

#### Triage
The question is probing into the assumptions about how data was laid out in both environments. Dev typically has smaller datasets and simplified configurations, while prod deals with full-scale data, which often involves complex skew patterns.

First-principles reasoning:

Partitioning Strategy: Check if the partitioning strategy differs between dev and prod. For instance, partitioning by date in dev might not capture the same level of granularity as in prod.
Data Skew: Examine whether data is skewed in prod but less so or non-existent in dev. Skewed data can lead to poor performance if not handled properly.
Dataset Size and Distribution: Dev datasets are often smaller, so skew might be more manageable, whereas prod has a much larger volume that could introduce significant skew.

#### Solution

Verify Partitioning: Ensure that the partition key used in dev is also effective for the full dataset in prod. For example, if date was partitioned in dev but not finely enough to handle prod’s scale, this could lead to suboptimal performance.
Optimize Data Layout: If skew is an issue, consider using a different partitioning strategy or implementing data re-partitioning logic within your Spark job. For instance, use dynamic partition pruning or rewrite queries to avoid hot partitions.
Tuning Spark Configurations: Adjust Spark configurations like spark.sql.shuffle.partitions and spark.sql.adaptive.enabled. These settings can help in managing large datasets more efficiently.

#### Closing Summary
and Common Mistakes:

The main assumptions that likely broke are around data distribution, partitioning strategy, and the handling of skewed data.

#### Common mistakes
include relying too heavily on dev settings without adequate testing for prod conditions or not addressing skew patterns adequately in the schema design.

**Comparison:**

| Environment | Sample Size | Partition Key   | Skew Observed |
|-------------|------------|----------------|--------------|
| Dev         | 10,000     | date           | Manageable    |
| Prod        | 1B         | date + category | Severe        |

**Code:**

```scala
// Example Spark job with dynamic partition pruning
val df = spark.read.table("production_table")
df.where($"date" >= "2023-01-01" and $"date" <= "2023-06-01").write.partitionBy("year", "month").parquet("/path/to/output")
```

**Example input / output:**

```text
Dev Environment:
- Sample size: 10,000 rows
- Partitioned by date (daily)
- Skew observed but manageable

Prod Environment:
- Full dataset: 1 billion rows
- Partitioned by date and category (monthly + yearly)
- Severe skew observed in certain categories
```

**Data:**

```json
{
  "dev_table_schema": {
    "fields": [
      {"name": "id", "type": "long"},
      {"name": "date", "type": "timestamp"},
      {"name": "value", "type": "double"}
    ]
  },
  "prod_table_schema": {
    "fields": [
      {"name": "id", "type": "long"},
      {"name": "date", "type": "timestamp"},
      {"name": "category", "type": "string"},
      {"name": "value", "type": "double"}
    ]
  }
}
```

**Flow:**

1. No
2. Check Partition Strategy
3. Is partition key same?
4. Verify dynamic partitionpruning
5. Adjust Spark config for prod
6. E
7. Re-run job in prod with tuning

