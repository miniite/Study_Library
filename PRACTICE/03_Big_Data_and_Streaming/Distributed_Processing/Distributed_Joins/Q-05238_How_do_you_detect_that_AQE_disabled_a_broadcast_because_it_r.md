---
type: "practice"
difficulty: "Deep"
---

# How do you detect that AQE disabled a broadcast because it read stale or missing statistics?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Verifying Join Strategy in Spark UI](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Verifying_Join_Strategy_in_Spark_UI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Detecting when AQE (Automatic Query Execution) disables a broadcast join due to stale or missing statistics involves monitoring query execution plans and understanding the conditions under which AQE decides to perform a shuffle join instead. This can be done by analyzing EXPLAIN output, examining optimizer settings, and leveraging diagnostic tools.

#### Triaging the Question

The question is asking about identifying situations where Apache Spark's AQE (Automatic Query Execution) decides not to use a broadcast join, which typically implies that either the statistics are stale or missing, leading the optimizer to choose a different execution plan. We need to understand how AQE works, what conditions lead to the decision-making process, and how we can detect these situations post-execution.

#### First-Principles Solution

Understanding Broadcast Joins:

A broadcast join is an efficient way of joining small datasets (broadcasted variables) with larger ones.
It's often used when one side of the join has a small number of unique keys, allowing for broadcasting that data to all nodes.

Automatic Query Execution (AQE):

AQE in Spark allows the optimizer to decide between different execution strategies based on runtime information such as statistics and partitioning.
It can dynamically choose between broadcast joins or shuffle joins depending on the current state of the cluster and the available data.

Monitoring AQE Decisions:

To detect when AQE disables a broadcast join due to stale statistics, we need to monitor how AQE makes its decisions during query execution.
This involves analyzing the EXPLAIN plans of queries and looking for specific patterns that indicate a shuffle join was chosen instead.

Analyzing EXPLAIN Plans:

Use EXPLAIN with options like VERBOSE, FORMAT JSON or FORMAT TEXT to get detailed information about the execution plan.
Look for hints in the plan such as "Shuffle Join" and "Broadcast Hash Join" nodes, comparing these with historical plans to spot changes.

Checking Statistics:

Ensure that your statistics are up-to-date and accurate. Use ANALYZE TABLE or DESCRIBE EXTENDED <table> to verify the statistics.
Monitor the stats collection process using tools like Spark UI (SQL tab) or specific metrics in cluster monitoring.

Using Diagnostic Tools:

Leverage Spark's diagnostics features such as spark.ui.retainedTime, spark.sql.statsLogging.enabled, and spark.sql.adaptive.minQuerySize.
Utilize the logging output to capture detailed information about query execution, including statistics updates and decisions made by AQE.

#### Closing Summary

To detect when AQE disables a broadcast join due to stale or missing statistics, one must analyze EXPLAIN plans, ensure accurate statistics, and use diagnostic tools. Regular monitoring of these aspects helps in identifying performance bottlenecks and optimizing query execution strategies.

#### Common Mistakes/Follow-Ups
Mistake: Ignoring the EXPLAIN plan details.

#### Follow-Up
Always review the full EXPLAIN output to understand why a particular join strategy was chosen.
Mistake: Not keeping statistics up-to-date.

#### Follow-Up
Schedule regular statistics collection and validation to avoid stale data issues.
Mistake: Overlooking diagnostic logs.

#### Follow-Up
Configure Spark UI and log settings for detailed insights into query execution.

**Comparison:**

| Column | Value |
|--------|-------|
| Key    | key_column |
| Type   | ShuffleJoinInner |

**Code:**

```python
# Example code snippet for analyzing EXPLAIN output using PySpark

from pyspark.sql import SparkSession

spark = SparkSession.builder.getOrCreate()

# Sample SQL query
df1 = spark.read.parquet("/path/to/data1")
df2 = spark.read.parquet("/path/to/data2")

result_df = df1.join(df2, "key_column", "inner")

# Explain the plan with verbose details
plan_explain = result_df.explain(plan="verbose")

print(plan_explain)
```

**Example input / output:**

```

Input:

SQL
Copy
SELECT * FROM table1 INNER JOIN table2 ON table1.key_column = table2.key_column;

Output:

CODE
Copy
EXPLAIN output (with verbose plan):
== Parsed Query ==
'broadcastHashJoin table1, table2 ON key_column [INNER]'

== Optimized Plan ==
ShuffleJoinInner
  Key: [key_column#1L]
  BuildSide: false
  OtherKey: [key_column#3L]
  Children:
    BroadcastHashJoin
      JoinType: Inner
      BuildKey: [key_column#1L]
      OtherKey: [key_column#3L]
      Input:
        table1
          TableScan: table1
    ShuffledRow
      Project: key_column#3L, other_columns_from_table2
```

**Data:**

```json
{
  "query_plan": {
    "type": "ShuffleJoin",
    "joinType": "INNER",
    "buildKey": ["key_column"],
    "otherKey": ["key_column"],
    "children": [
      {"id": 1, "name": "BroadcastHashJoin"},
      {"id": 2, "name": "ShuffledRow"}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Is AQE Enabled?
7. Analyze EXPLAIN Plan
8. Check Statistics Collection
9. Determine Join Strategy Basedon Plan
10. Verify Statistics Accuracy
11. No Action Required
12. Run ANALYZE TABLE
13. Monitor Performance

