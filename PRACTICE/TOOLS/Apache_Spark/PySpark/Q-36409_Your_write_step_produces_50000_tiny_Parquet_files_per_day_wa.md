---
type: "practice"
difficulty: "Deep"
---

# Your write step produces 50000 tiny Parquet files per day — walk me through the small-files problem and how you'd fix it upstream in the Spark job.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reducing Shuffle in PySpark](<../../../../TOOLS/Apache_Spark/PySpark/Reducing_Shuffle_in_PySpark.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a large number of small files like Parquet files, we encounter the "small-files problem," which can lead to inefficiencies in Spark jobs due to increased overhead from file system operations and context switching between tasks. To address this issue, we need to aggregate these small files into larger partitions or reduce their count.

To fix this upstream in the Spark job, we can employ several strategies:

Increase Partition Size: Adjust the partitioning logic to group records more coarsely.
Coalesce Operations: Use coalesce to reduce the number of partitions after a transformation.
Combine Multiple Files: Aggregate multiple small files into larger ones using union or similar operations.

Let's dive deeper into these strategies and how they can be implemented in PySpark.

