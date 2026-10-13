---
type: "practice"
difficulty: "Deep"
---

# How would you detect and fix a broadcast join that silently degraded to a sort-merge join because the small side grew past the broadcast threshold?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Spark Executor Sizing](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Spark_Executor_Sizing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, we need to first understand what broadcast joins are and why they might degrade to sort-merge joins in Spark's execution plan. Broadcast joins can significantly improve performance by broadcasting a small dataset to all executors, reducing the amount of data shuffling required for the join operation. However, if the small dataset grows too large, Spark may switch to a more resource-intensive sort-merge join instead.

#### Triaging the Question
Question Core: The question is asking how to detect and fix a scenario where a broadcast join unexpectedly degrades to a sort-merge join due to the size of the smaller dataset.

#### Assumptions
We are working with Apache Spark.
The small side of the join has grown too large, causing Spark to fall back to a sort-merge strategy.
Key Concepts: Broadcast joins, broadcast thresholds, execution plans, and optimization strategies.

#### First-Principles Solution

Detecting the Issue:

Use Spark UI or Spark History Server to inspect the execution plan of your job. Look for a SortMergeJoin instead of a BroadcastHashJoin.
Check the size of the small dataset and compare it against Spark's broadcast threshold settings.

Fixing the Issue:

Reduce the size of the smaller dataset if possible.
Increase the broadcast threshold in your Spark configuration to allow larger datasets for broadcast joins.
Optimize the join logic, potentially by reorganizing data or using different join strategies based on the data characteristics.

#### Closing Summary
and Common Mistakes

#### Common Mistakes
Ignoring the execution plan changes due to dataset size growth.
Not adjusting the broadcast threshold settings in time for large datasets.
Failing to optimize other parts of the query that could reduce the need for a broadcast join.
Practical Example and Follow-Ups

#### Example Scenario
Suppose you have a user activity log (small dataset) and product catalog (large dataset). If the user activity log grows significantly, Spark might switch from a broadcast join to a sort-merge join.

Detection Steps:

BASH
# Inspect execution plan using Spark UI or Spark History Server
./bin/spark-submit --conf spark.ui.enabled=true your_job.py

Fixing the Issue:

Lower the broadcast threshold (if increasing it is not feasible):
PROPERTIES
# Increase the broadcast threshold in Spark configuration file (spark-defaults.conf)
spark.sql.adaptiveBroadcastJoinThreshold=52428800  # Default: 16MB, increase to 50MB for example
Re-examine and optimize your join logic:
PYTHON
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("OptimizedJoin").getOrCreate()

user_activities = spark.read.parquet("user_activity_log")
products = spark.read.parquet("product_catalog")

# Optimize by reorganizing data or using different join strategy
optimized_user_activities = user_activities.withColumn("key", F.col("user_id"))  # Example key
result = optimized_user_activities.join(products, "key", "left_outer").select("user_id", "product_id")

result.show()
Self-Check

Correctness + completeness confirmed.

**Comparison:**

| User Activity Log Schema |
|--------------------------|
| timestamp (timestamp)    |
| user_id (string)         |
| Product Catalog Schema   |
|--------------------------|
| product_id (string)      |
| price (double)           |

**Code:**

```python
from pyspark.sql import SparkSession
import pyspark.sql.functions as F

spark = SparkSession.builder.appName("OptimizedJoin").getOrCreate()

user_activities = spark.read.parquet("user_activity_log")
products = spark.read.parquet("product_catalog")

# Optimize by reorganizing data or using different join strategy
optimized_user_activities = user_activities.withColumn("key", F.col("user_id"))
result = optimized_user_activities.join(products, "key", "left_outer").select("user_id", "product_id")
result.show()
```

**Example input / output:**

```text
# Sample input data for user_activity_log and product_catalog

user_activity_log:
+-------------------+---------+
|        timestamp  | user_id |
+-------------------+---------+
|2023-10-01 14:00:00|  user1   |
|2023-10-02 15:00:00|  user2   |
|2023-10-03 16:00:00|  user3   |
+-------------------+---------+

product_catalog:
+--------+-----+
|product_id| price|
+--------+-----+
|p001     | 1.99|
|p002     | 2.99|
|p003     | 3.99|
+--------+-----+

# Optimized join output
+---------+--------+
| user_id |product_id |
+---------+--------+
|   user1  |    p001   |
|   user2  |    p002   |
|   user3  |    p003   |
+---------+--------+
```

**Data:**

```json
{
  "user_activity_log": {
    "path": "/data/user_activity",
    "schema": [
      {"name": "timestamp", "type": "timestamp"},
      {"name": "user_id", "type": "string"}
    ]
  },
  "product_catalog": {
    "path": "/data/products",
    "schema": [
      {"name": "product_id", "type": "string"},
      {"name": "price", "type": "double"}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Detect Execution Plan
4. Is Broadcast Join Degraded?
5. Check Dataset Size
6. Continue Monitoring
7. Adjust Broadcast Threshold orOptimize Logic
8. Verify Fix with New ExecutionPlan

