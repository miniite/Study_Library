---
type: "practice"
difficulty: "Intermediate"
---

# What's the difference between logical and physical operator IDs in the job graph and why does that matter when reading the Flink UI during an incident?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Keyed vs Operator State](<../../../../TOOLS/Apache_Flink/Flink_Core/Keyed_vs_Operator_State.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Apache Flink, a stream processing job is visualized as a directed acyclic graph (DAG) of operators. Logical operator IDs are high-level identifiers that represent operations in your data flow, while physical operator IDs are lower-level identifiers that map to actual instances on the cluster's tasks. Understanding these differences can help you diagnose issues more effectively when using the Flink UI during an incident.

To dive into this concept, let’s break it down step by step:

Logical Operators vs Physical Operators:

Logical Operator IDs: These are conceptual identifiers used in your job’s logical plan. They represent high-level operations like map, filter, and join. For example, if you have a map operation that doubles the value of each element, it might be represented by a logical operator ID.
Physical Operator IDs: These correspond to actual tasks on the cluster. Each physical operator is an instance of a logical operator, assigned a unique identifier in the Flink UI. This can help pinpoint specific parts of your job execution.

Why It Matters During Incidents:

When you encounter issues during a stream processing job, understanding these IDs helps you locate and diagnose problems more precisely. For example, if an exception occurs at a particular physical operator ID, it indicates where the failure happened in the data flow.
The logical operators provide a high-level view of your application’s structure, making them easier to understand and reason about. Physical operators offer detailed insights into specific task execution.

First-Principles Explanation:

A Flink job starts with a stream or a source (like Kafka), which is transformed through various logical operations like map, filter, reduce, etc., before being written to a sink.
Each of these operations in the logical plan gets an ID. When you run your job on a cluster, each instance of these operations becomes a physical operator with its own task IDs.

#### Example Scenario

Imagine you have a Flink job that processes real-time financial transactions, filtering out invalid ones and then aggregating volumes by stock symbol.
The logical plan might look like this:
PLAINTEXT
(transactionStream -> filter -> map -> aggregate)
In the Flink UI, these operations will be represented as physical operators with their own IDs. If an error occurs during the filter step, you can quickly identify which physical operator is responsible.

#### Closing Summary
and Common Mistakes:

Logical operators provide a clear view of your application’s architecture, making them useful for debugging at a high level.
Physical operators are crucial for pinpointing specific task failures. Misunderstanding the difference between these IDs can lead to inefficiencies in troubleshooting issues.

