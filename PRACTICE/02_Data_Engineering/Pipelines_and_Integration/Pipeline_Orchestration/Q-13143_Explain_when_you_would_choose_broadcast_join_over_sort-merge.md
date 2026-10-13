---
type: "practice"
difficulty: "Deep"
---

# Explain when you would choose broadcast join over sort-merge join in Spark for a feature pipeline and what happens if the broadcast side is too large.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Bucketed Joins in Spark](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Bucketed_Joins_in_Spark.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In data pipelines, particularly with Spark, choosing between broadcast join and sort-merge join depends on the size of your datasets and the specific use case. Broadcast joins are highly efficient when the smaller dataset (the "broadcast" side) can fit entirely into memory on a single node. Conversely, sort-merge joins handle large datasets by merging them after sorting both sides. Let's dive into each scenario from first principles.

First, we need to understand what these joins do:

Broadcast Join: The smaller table is loaded into memory in a worker node and then broadcasted across all nodes where the join operation takes place.
Sort-Merge Join: Both datasets are sorted and then merged on the fly during the execution. This works well for large datasets but requires additional sorting overhead.

To decide, consider the size of your data:

Broadcast Join is ideal when the smaller dataset can fit into memory (typically under 2 GB). It’s very fast because it avoids the costly sorting step and uses less network bandwidth.
Sort-Merge Join is necessary for larger datasets where broadcasting might exceed memory limits, making in-memory broadcast impractical.

Now, let's apply this to a feature pipeline: Imagine you have a large user dataset (users) and a smaller category mapping table (categories). For each user, you need to map them to their corresponding categories. If the categories table is small enough (e.g., 1000 rows), using a broadcast join would be highly efficient because it can fit into memory.

However, if your categories table grows significantly larger, say to millions of rows, broadcasting becomes infeasible due to memory constraints. In such cases, sort-merge joins are the better choice as they handle large datasets more gracefully by sorting and merging them on the fly.

If the broadcast side is too large:

Memory Overflow: The Spark job will fail with an out-of-memory (OOM) error.
Performance Degradation: Even if it doesn’t cause a crash, the performance might degrade significantly as Spark tries to handle very large datasets in memory.

To mitigate this issue, consider these strategies:

Increase Memory: If possible, allocate more memory to your worker nodes.
Data Partitioning: Split the larger table into smaller chunks that can fit into memory and use broadcast join for each chunk.
Optimize Data: Reduce the size of the data by filtering or aggregating where necessary.

In conclusion, the choice between broadcast join and sort-merge join hinges on dataset sizes. For small datasets (<2 GB), broadcast join is preferred due to its speed and efficiency. For larger datasets, sort-merge joins ensure robust performance without memory constraints.

**Comparison:**

| Join Type     | When to Use                     | Pros                                    | Cons                                  |
|---------------|---------------------------------|-----------------------------------------|---------------------------------------|
| Broadcast Join | Smaller dataset fits memory     | Fast, efficient, less network overhead  | Memory limit constraints              |
| Sort-Merge Join | Larger datasets                 | Handles large datasets                   | Additional sorting overhead            |

**Code:**

```python
# Example of a Spark job using both joins
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("JoinExample").getOrCreate()

users_df = spark.read.csv("path/to/users.csv", header=True, inferSchema=True)
categories_df = spark.read.csv("path/to/categories.csv", header=True, inferSchema=True)

# Small dataset - Broadcast Join
small_categories_df = categories_df.repartition(1)  # Ensure small enough to fit in memory
joined_df = users_df.join(small_categories_df, on="category_id", how="left_outer")

# Large dataset - Sort-Merge Join
large_categories_df = categories_df.repartition("category_id")  # Partition by a key column
joined_large_df = users_df.join(large_categories_df, on="category_id", how="left_outer")
```

**Example input / output:**

```text
Input:
users_df = ...
small_categories_df = ...

Output (Broadcast Join):
+----+-------+-----------+
|user_id|name|category_description|
+----+-------+-----------+
|1    |Alice |Tech       |
|2    |Bob   |Science    |
|3    |Charlie|Math      |
+----+-------+-----------+

Input:
users_df = ...
large_categories_df = ...

Output (Sort-Merge Join):
+----+-------+-----------+
|user_id|name|category_description|
+----+-------+-----------+
|1    |Alice |Tech       |
|2    |Bob   |Science    |
|3    |Charlie|Math      |
+----+-------+-----------+
```

**Data:**

```json
{
  "users": {
    "user_id": [1, 2, 3],
    "name": ["Alice", "Bob", "Charlie"],
    "category_id": [101, 102, 103]
  },
  "categories": {
    "category_id": [101, 102, 103],
    "description": ["Tech", "Science", "Math"]
  }
}
```

**Flow:**

1. Yes
2. No
3. Check Dataset Size
4. Is Small (<2GB)?
5. Broadcast Join
6. Sort-Merge Join
7. Highly Efficient, Fast Execution
8. Handles Large Datasets,Robust

