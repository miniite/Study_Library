---
type: "practice"
difficulty: "Basic"
---

# Why is Spark called a lazy engine and what benefit does lazy evaluation give when building an ETL job?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Shuffle Partition Tuning](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Shuffle_Partition_Tuning.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Spark, often referred to as a "lazy engine," processes data in a lazy manner by deferring computation until it absolutely needs to. This approach is particularly useful in big data processing scenarios where dealing with massive datasets can be computationally expensive and time-consuming.

The core concept of lazy evaluation means that Spark doesn’t execute the transformations (like filtering, mapping) immediately when you define them; instead, it creates an “abstract syntax tree” or a Directed Acyclic Graph (DAG). Only when you start to action the final output does Spark actually start executing those operations. This is very beneficial for ETL jobs as it allows efficient management of large datasets and optimizes resource utilization.

To illustrate this with an example, imagine we have a huge dataset that we want to filter based on certain conditions. With lazy evaluation, Spark will not perform any computation until the final output is requested. It just sets up the plan in memory, waiting for the execution trigger.

**Example input / output:**

```

In the above example:

Input: A Spark DataFrame with columns id and name.
Output: A transformation step that filters rows where the name is "Alice", but no actual computation occurs yet.
PLAINTEXT
Copy
Input:
+---+-----+
| id| name|
+---+-----+
|  1|Alice|
|  2|  Bob|
|  3|Charlie|
+---+-----+

Transformation (not executed yet):
filtered_df = df.filter(df["name"] == "Alice")
```

