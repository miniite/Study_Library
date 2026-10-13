---
type: "practice"
difficulty: "Intermediate"
---

# Describe how you'd read the Spark UI to confirm whether a join actually broadcast or fell back to shuffle.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Verifying Join Strategy in Spark UI](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Verifying_Join_Strategy_in_Spark_UI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To determine if a join operation in Apache Spark is using a broadcast join, I would look at the Spark UI to analyze the execution plan and metrics of my job. Essentially, I’m looking for evidence that Spark chose to use a broadcast join over a shuffle-based join. Broadcast joins are efficient when one dataset (the smaller one) can fit entirely into memory on each executor, allowing Spark to distribute it efficiently.

#### Triaging the Question

The core of this question is understanding how to interpret Spark UI data to diagnose join operations. The key aspects are:

Identifying whether a broadcast join was used: This involves checking the execution plan.
Checking for any signs that a shuffle join was attempted and failed: This could indicate performance issues or incorrect configurations.

#### Assumptions
You have access to Spark UI, specifically the job details page.
The datasets involved in the join are known (e.g., their sizes).
The data is reasonably large for meaningful comparison between broadcast and shuffle joins.

#### First-Principles Solution
Step 1: Accessing Spark UI

First, navigate to the Spark application's UI by going to http://<driver-node>:4040 or using a cluster manager like Apache Mesos, YARN, or Kubernetes. From there, select the job you want to analyze.

Step 2: Examining the Execution Plan

In the job details page, scroll down to the "Stages" tab. Here, you can see the individual stages of your job and their associated tasks. Each stage corresponds to a single shuffle operation or broadcast join.

Broadcast Join: Look for an entry in the execution plan that says "Shuffle Binner -> BroadcastHashJoin". This indicates Spark has chosen to perform a broadcast hash join.
Shuffle Join: Look for entries like "Shuffle Reader -> ShuffleWriter" or "ShuffledKey->ReduceTask". These indicate Spark is performing a shuffle-based join.
Step 3: Analyzing Metrics

Once you have identified the relevant tasks, click on them to view detailed metrics:

Broadcast Size: If a broadcast join was used, the size of the broadcasted data should be shown here.
Shuffle Data Size: For shuffle joins, the amount of shuffled data and the number of shuffles will be displayed.
Step 4: Comparing Join Types

Compare the sizes and times for broadcast vs. shuffle operations:

A broadcast join should generally have smaller shuffle data sizes and faster completion times if the dataset fits into memory.
Shuffle joins might show larger shuffle data sizes, more shuffles, and slower execution times.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, to confirm whether a join operation is using a broadcast or fallbacks to a shuffle, carefully examine the Spark UI's execution plan and metrics. Broadcast joins are typically indicated by reduced shuffle operations and smaller broadcasted dataset sizes.

#### Common mistakes
might include not checking for the specific keywords in the execution plan ("BroadcastHashJoin") or misinterpreting the metrics (e.g., assuming larger shuffled data always means a bad join when it might be necessary).

For follow-ups, you could discuss:

How to optimize the datasets for broadcast joins.
The trade-offs between broadcast and shuffle joins based on dataset sizes.
Situations where a fallback to a shuffle join is still beneficial.

**Comparison:**

| Task ID | Stage ID | Details                |
|---------|----------|------------------------|
| 0       | 123456   | Shuffle Binner -> BroadcastHashJoin |
| 1       | 123456   | ShuffledKey->ReduceTask    |

**Code:**

```python
# Example Python code snippet to parse job metrics (hypothetical)
from pyspark.sql import SparkSession

spark = SparkSession.builder.getOrCreate()
job_id = 123456
# Fetch and parse the execution plan and metrics from Spark UI API or logs
execution_plan = spark._jvm.org.apache.spark.scheduler.ui.SchedulerUI.getExecutionPlan(job_id)
metrics = spark._jvm.org.apache.spark.scheduler.ui.StageDetailsMetrics()

# Print relevant parts of the execution plan
print(execution_plan)
print(metrics.getShuffleDataSize())
```

**Example input / output:**

```text
Input:
- Join operation on two datasets, where the smaller dataset is expected to be broadcasted.

Output:
- Execution plan shows "Shuffle Binner -> BroadcastHashJoin" for one task and "ShuffledKey->ReduceTask" for another.
- Metrics indicate small shuffle data size and fast completion time for the broadcast join task.
```

**Data:**

```json
{
  "stage": {
    "name": "Stage 123456",
    "status": "SUCCEEDED",
    "tasks": [
      {
        "task_id": 0,
        "stage_id": 123456,
        "attempt_id": 0,
        "is_final_status": true,
        "details": "Shuffle Binner -> BroadcastHashJoin"
      },
      {
        "task_id": 1,
        "stage_id": 123456,
        "attempt_id": 0,
        "is_final_status": true,
        "details": "ShuffledKey->ReduceTask"
      }
    ]
  }
}
```

**Flow:**

1. Access Spark UI
2. Select Job
3. View Execution Plan
4. Identify 'Shuffle Binner ->BroadcastHashJoin'
5. Check Shuffle Data Metrics
6. Compare Join Types
7. Confirm Broadcast Join

