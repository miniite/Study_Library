---
type: "practice"
difficulty: "Intermediate"
---

# How do you set spark.sql.autoBroadcastJoinThreshold to -1 safely and what's the blast radius of that change?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Verifying Join Strategy in Spark UI](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Verifying_Join_Strategy_in_Spark_UI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When tuning Spark jobs, setting spark.sql.autoBroadcastJoinThreshold to -1 can significantly optimize query performance by eliminating broadcast joins altogether. However, this approach has its risks and should be carefully considered before deployment.

#### Triage

The question is about understanding the implications of setting spark.sql.autoBroadcastJoinThreshold to -1. This value controls when Spark will automatically apply a broadcast join between two data sets. Setting it to -1 means that no broadcast joins will occur, which can lead to different execution plans and may not always be beneficial.

#### Assumptions
We are working with Apache Spark.
The job involves multiple data frames or datasets joined together.
There is a need for performance tuning in an existing Spark application.

#### First Principles Solution

Understanding Broadcast Joins: A broadcast join is useful when one of the smaller data sets can be stored in memory on each executor, thus reducing shuffling and improving performance.

Setting spark.sql.autoBroadcastJoinThreshold to -1: This setting means that Spark will never use a broadcast join regardless of the size of the datasets involved. Instead, it will always perform a shuffle-based join.

Implications:

Performance Impact: Broadcast joins are beneficial when one dataset is small enough to fit in memory on all executors. Disabling them can lead to inefficient shuffling and increased latency.
Resource Usage: Enabling broadcast joins for large datasets can consume more executor memory, leading to out-of-memory errors if the data exceeds available resources.

Blast Radius:

Job Performance: The change might negatively impact the performance of all jobs running with this setting, especially those that benefit from broadcast joins.
Memory Usage: It could lead to increased memory usage across the cluster, as larger datasets are joined without broadcasting.
Latency: Jobs that were previously optimized for small broadcasts might experience higher latencies due to shuffling.

Safe Deployment:

Test Thoroughly: Run comprehensive tests in a staging environment before applying this change in production.
Monitor Performance: After deployment, monitor the performance metrics to ensure there are no unexpected issues.
Backup Configuration: Keep a backup of the previous configuration settings and revert if necessary.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Setting spark.sql.autoBroadcastJoinThreshold to -1 is a powerful tool for performance tuning but should be used cautiously. The blast radius can affect not just one job, but potentially all jobs running on the cluster. It’s crucial to thoroughly test and monitor after making such changes.

#### Common Mistakes
Failing to understand the underlying mechanics of broadcast joins.
Applying this setting without testing in a controlled environment.
Not considering the resource implications for larger datasets.

#### Follow-up
questions might include:

How do you handle memory constraints when using large datasets?
What are some alternatives to broadcast joins if they are not suitable?

**Code:**

```python
# Example code snippet to set the configuration in Spark session
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", -1)
```

