---
type: "practice"
difficulty: "Deep"
---

# Walk me through why a query plan shows an unexpected Cartesian product and how you would fix it.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Shuffle Tuning and Broadcast Joins](<../../../../TOOLS/Apache_Spark/Spark_Core/Shuffle_Tuning_and_Broadcast_Joins.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview
A Cartesian product in Apache Spark can occur when a join operation is performed without proper conditionals, leading to every row from one DataFrame being paired with every row from another. This results in an exponentially large dataset that can be very inefficient and resource-intensive. To resolve this issue, you need to ensure the correct join conditions are applied.

#### Triage
The question is asking about understanding and troubleshooting unexpected Cartesian products in Spark query plans. We'll assume the Spark version used supports these optimizations and the dataframes involved have matching schemas for potential joins.

Identify the Problem: An unexpected Cartesian product indicates that a Cartesian or CrossJoin operation was performed instead of an intended join.
Understand Join Types: In Spark, a typical join (like Inner, Left_outer, Right_outer) should not produce a Cartesian product unless explicitly called for. The issue usually arises from incorrect usage of the crossJoin() method without appropriate conditions.
Review Query Plan: Use explain to examine the plan and identify where the Cartesian operation is being applied. Look for operations like CrossJoin that indicate an unexpected join.

Step-by-Step Solution:

Examine the Spark SQL Code:

SCALA
val df1 = spark.read.parquet("path/to/data1")
val df2 = spark.read.parquet("path/to/data2")

// Incorrect join without conditions
val cartesianResult = df1.crossJoin(df2)

// Correct join with conditions
val correctResult = df1.join(df2, df1.col("id") === df2.col("id"), "inner")

Analyze the Explain Plan:

SCALA
cartesianResult.explain(true)  // This will show a Cartesian product
correctResult.explain(true)    // This should show an efficient join plan

Correct the Join Conditions: If you need to use a cross join, ensure there's a valid reason for it:

SCALA
val crossJoined = df1.crossJoin(df2)

Ensure this aligns with your business logic.

Optimize Joins and Operations:

Use appropriate partitioning and indexing.
Optimize data schemas to reduce the need for complex joins.

#### Cons
ider using broadcast joins if one DataFrame is small enough:
SCALA
val result = df1.join(broadcast(df2), df1.col("id") === df2.col("id"), "inner")

#### Closing Summary
By carefully examining the join conditions and query plans, you can avoid unexpected Cartesian products. Always use appropriate join types based on your data requirements and ensure that each join is necessary and efficient.

#### Common Mistakes

Misusing crossJoin() without proper conditions.
Overlooking the importance of explain plans in identifying suboptimal queries.
Ignoring schema optimization to reduce unnecessary operations.

#### Follow-Up
How would you optimize this query further if both DataFrames are large? Would partitioning help?

**Code:**

```scala
val df1 = spark.read.parquet("path/to/data1")
val df2 = spark.read.parquet("path/to/data2")

// Incorrect join without conditions
val cartesianResult = df1.crossJoin(df2)
cartesianResult.explain(true)

// Correct join with conditions
val correctResult = df1.join(df2, df1.col("id") === df2.col("id"), "inner")
correctResult.explain(true)
```

**Example input / output:**

```text
// Explain Plan for Incorrect Join
== Physical Plan ==
CrossJoin
:- BroadcastHashJoin [id#35L], [id#98L], Inner, false
:- BroadcastExchange HashedRelationBroadcastMode(List(lit(1)))
+- LocalTableScan [name#34L]
+- Exchange rangepartitioning(id#98L ASC), batchId=0
   +- LocalTableScan [value#97L]

// Explain Plan for Correct Join
== Physical Plan ==
ShuffledHashJoin [id#35L = id#98L], Inner, false
:- LocalTableScan [name#34L]
+- Exchange rangepartitioning(id#98L ASC), batchId=0
   +- LocalTableScan [value#97L]
```

**Data:**

```json
{
  "path/to/data1": {
    "schema": ["id", "name"],
    "data": [
      {"id": 1, "name": "A"},
      {"id": 2, "name": "B"}
    ]
  },
  "path/to/data2": {
    "schema": ["id", "value"],
    "data": [
      {"id": 1, "value": 10},
      {"id": 2, "value": 20}
    ]
  }
}
```

