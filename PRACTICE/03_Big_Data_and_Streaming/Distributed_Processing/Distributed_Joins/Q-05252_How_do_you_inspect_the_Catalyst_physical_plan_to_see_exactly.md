---
type: "practice"
difficulty: "Intermediate"
---

# How do you inspect the Catalyst physical plan to see exactly which exchange and join operators will run?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Join Strategies in Spark](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Join_Strategies_in_Spark.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand which exchange and join operators will run in a Spark job, we need to inspect the Catalyst physical plan. The Catalyst optimizer generates this plan from the logical plan, detailing how data will be processed at runtime. This allows us to pinpoint exactly where shuffles and broadcasts occur.

The question is asking you to use Spark's APIs to visualize the execution plan and identify specific operators like BroadcastShuffleJoin or ShuffleExchange. You should assume familiarity with Spark 3.x and its Catalyst optimizer. The key idea is to leverage explain methods on DataFrame/Dataset operations to get a clear breakdown of the physical plan.

Let’s break down the steps:

Understand Logical vs Physical Plan: First, recall that the logical plan represents how you wrote your transformations in Spark, while the physical plan shows how these will be executed by the runtime.
Use explain Methods: You can use explain(true) on DataFrame/Dataset operations to get a detailed textual representation of the execution plan. For a full breakdown, including specific operators like BroadcastShuffleJoin or ShuffleExchange, you might need to use explainextended.
Analyze the Plan: Look for operators that indicate shuffles and broadcasts. Common operators include ShuffledHashJoin, BroadcastHashJoin, and BroadcastExchange.

Here is how you would approach this from first principles:

Identify Join Operations: Start by performing a join operation on your DataFrame/Dataset.
Invoke Explain Methods:
PYTHON
df.explain(True)
Analyze the Output: The output will give you a detailed view of how Spark plans to execute this operation.

Let’s walk through an example:

#### Example
Code and Explanation
PYTHON
from pyspark.sql import SparkSession

# Initialize Spark session
spark = SparkSession.builder.appName("BroadcastShuffleJoinExample").getOrCreate()

# Sample DataFrames
df1 = spark.createDataFrame([(1, 'a'), (2, 'b')], ['id', 'value'])
df2 = spark.createDataFrame([(1, 10), (3, 20)], ['id', 'amount'])

# Perform a join operation
joined_df = df1.join(df2, on='id')

# Explain the plan in detailed mode
joined_df.explain(True)
Output Analysis

The output will look something like this:

PLAINTEXT
== Physical Plan ==
ShuffledHashJoin [id#0L], [id#3L], BuildRight
+- Exchange rangepartitioning(id#0L, 200), ENSURE_DISTRIBUTE
   +- Scan ExistingRDD[id#0L,value#1L]
+- BroadcastExchange IdentityBroadcastExchange(V_#4)
   +- Scan ExistingRDD[id#3L,amount#5L]

In this output:

ShuffledHashJoin indicates a shuffle join.
Exchange rangepartitioning shows the partitioning strategy used by Spark for shuffling.
BroadcastExchange indicates that a broadcast join is being used.

#### Closing Summary
and Common Mistakes

To summarize, understanding which exchange and join operators will run involves using explain methods on DataFrame/Dataset operations. The key is to look for specific operator names like ShuffledHashJoin, BroadcastHashJoin, and BroadcastExchange.

#### Common mistakes

Forgetting to use the correct explain method (explain(true) or explainextended()).
Not recognizing the subtle differences between various join operators.

By following these steps, you can effectively inspect the Catalyst physical plan to identify shuffles and broadcasts in your Spark job.

