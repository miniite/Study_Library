---
type: "practice"
difficulty: "Deep"
---

# How do you reason about consistency guarantees a business actually needs: read-your-writes, monotonic reads, eventual? Map those to unified architecture choices and where over-engineering hurts.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Unified Batch and Streaming Semantics](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Batch_and_Stream_Reconciliation/Unified_Batch_and_Streaming_Semantics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with batch and stream unification in data engineering, the core challenge is ensuring that a business has the appropriate consistency guarantees. This involves understanding what consistency means (read-your-writes, monotonic reads, eventual) and mapping these requirements to architectural choices. Over-engineering can lead to unnecessary complexity and cost, so it's crucial to strike a balance.

The question essentially probes your ability to:

Understand different levels of consistency.
Map those guarantees to specific use cases and data flow scenarios.
Design a unified architecture that balances performance, cost, and reliability.
Avoid over-engineering by ensuring the solution matches business needs precisely.

#### Assumptions

The business requires reliable data processing for decision-making.
There are varying levels of real-time requirements (from immediate to eventually consistent).
Cost is a significant factor in design decisions.
Step-by-Step Solution

Identify Business Requirements:

Understand the key stakeholders and their needs.
Determine critical business processes that depend on data consistency.

Map Consistency Guarantees:

Read-your-writes: Ensure that a write operation is visible to subsequent reads by the same user or session.
Monotonic reads: Ensure that once a value changes, all future reads see a non-decreasing sequence of values.
Eventual consistency: The system will reach a state where all operations succeed and no further updates occur.

Analyze Data Flow:

Batch Processing: Typically involves historical data and reporting needs.
Stream Processing: Deals with real-time or near-real-time data streams.

Unified Architecture Choices:

Use Apache Flink for both batch and stream processing, as it supports stateful computations and can handle both modes efficiently.
Kafka Streams for stream processing, leveraging the same Kafka cluster for storage and stream management.

Designing for Read-Your-Writes:

Implement snapshot isolation or MVCC (Multi-Version Concurrency Control) to ensure reads are consistent with writes by the same session.
Use transactional semantics in Flink jobs if required.

Monotonic Reads:

For time-series data, maintain a strict ordering of updates.
Use KTable and KStream APIs in Kafka Streams for maintaining stateful computations that respect order.

Eventual Consistency:

Leverage Apache Pulsar or AWS Kinesis for event sourcing.
Design applications to handle eventual consistency by implementing retries, backoffs, and fallback strategies.

Avoiding Over-Engineering:

Evaluate whether real-time processing is truly necessary. Often, periodic batch jobs can serve the same purpose with lower costs.

#### Cons
ider caching mechanisms like Redis or Memcached for read-heavy workloads to reduce database load without compromising consistency guarantees.

Cost and Scalability:

Choose cost-effective storage solutions based on data access patterns (e.g., S3 for infrequently accessed historical data).
Design scalable architectures that can handle spikes in real-time data volume.

#### Closing Summary

To achieve a unified architecture for batch and stream processing, it's essential to map business requirements to specific consistency guarantees. By using technologies like Flink and Kafka Streams, you can design systems that meet read-your-writes, monotonic reads, or eventual consistency needs while minimizing over-engineering. Careful consideration of cost, scalability, and real-time requirements ensures a balanced solution.

#### Common Mistakes/Follow-Ups
Overlooking the importance of understanding business-specific needs.
Ignoring trade-offs between consistency levels and performance.
Failing to consider cost implications of real-time vs. batch processing choices.
Not leveraging caching or stateful processing effectively, leading to unnecessary complexity in stream architectures.

**Comparison:**

```
Consistency Level	Description	Use Case Example
Read-your-writes	Reads always see the result of a write by that session.	User profile updates seen immediately in all pages.
Monotonic reads	Values are non-decreasing as they evolve over time.	Stock prices tracked over time, ensuring no downgrades.
Eventual	Guarantees eventual agreement but not immediate visibility.	Real-time trading system with periodic reconciliation.
```

**Code:**

```java
// Example Flink job setup for read-your-writes
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
DataStream<String> input = env.readTextFile("/path/to/input");

input.keyBy((String record) -> extractKey(record))
   .flatMap(new MyStatefulFunction())
   .writeAsText("/path/to/output");
```

**Example input / output:**

```text
Input:
- Batch job reads historical data from S3.
Output:
- Updated reports generated and stored in an S3 bucket.

Input:
- Stream processing updates a time-series database with new sensor readings.
Output:
- Real-time alerts triggered based on threshold values, stored in Kafka.
```

**Data:**

```json
{
  "data": [
    {"id": 1, "value": 20},
    {"id": 2, "value": 30}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Batch Job
4. S3 Input
5. Read-Your-Writes?
6. Write to S3
7. Eventual Consistency withKafka
8. Kafka Output

