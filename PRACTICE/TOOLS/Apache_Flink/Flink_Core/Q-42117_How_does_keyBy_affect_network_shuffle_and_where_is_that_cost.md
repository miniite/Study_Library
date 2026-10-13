---
type: "practice"
difficulty: "Deep"
---

# How does keyBy affect network shuffle, and where is that cost visible in a high-cardinality streaming aggregation?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Keyed vs Operator State](<../../../../TOOLS/Apache_Flink/Flink_Core/Keyed_vs_Operator_State.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

KeyBy in Apache Flink significantly impacts how data is distributed across parallel tasks during stream processing. It determines which keys from the input data are used to assign the elements into different partitions, thereby influencing whether network shuffles occur and their associated costs. In this answer, we will delve into how KeyBy works, its impact on network shuffle, and where you can observe these effects in high-cardinality streaming aggregations.

#### Triage
What is KeyBy?

KeyBy is a fundamental operation in Apache Flink that allows users to partition the input stream based on specific keys. This operation is crucial for implementing stateful operations such as windowed aggregations or joins. The choice of key influences how data is distributed among parallel instances, which can affect performance.

Step-by-Step Solution: Understanding KeyBy and Network Shuffle
Key Assignment: When you apply keyBy to a stream, Flink assigns each incoming element a partition ID based on the provided key function. This function extracts one or more fields from an element to determine its partition.
Partitioning Strategy: Flink uses consistent hashing to map keys to partitions. For high-cardinality keys (i.e., many unique values), this strategy ensures that data is evenly distributed among partitions, minimizing hotspots and load imbalances.
Network Shuffle: If the key function results in a large number of distinct keys, it can lead to significant network shuffles. This occurs when elements with different keys are sent over the network to different tasks for processing. The cost of network shuffle is high because it involves serialization, deserialization, and network communication overhead.
Detailed Analysis: High-Cardinality Streaming Aggregation

#### Example Scenario
Consider a stream of log entries where each entry has an userId field. If there are millions of unique userIds (high cardinality), applying keyBy(userId) will result in a large number of partitions.
Network Shuffle Cost: Each log entry is hashed to determine its partition, and if the hash values span many partitions, the entries may need to be shuffled across the network multiple times. This leads to increased network traffic and processing delays.
Observing Network Shuffle Costs
Flink Web UI: You can observe the impact of network shuffle in the Flink Web UI by checking the "Job" tab. Look for metrics such as "Shuffle Input Bytes" and "Shuffle Output Bytes". High values indicate significant network activity.
Monitoring Tools: Use external monitoring tools like Prometheus or Grafana to visualize network traffic related to your Flink job. Monitoring network.in.bytes and network.out.bytes can help identify where the costs are high.

#### Trade-Offs
Consistency vs. Cost

#### Cons
istency Requirement: In some cases, high-cardinality keys may be necessary for achieving data consistency across tasks. For instance, in a real-time recommendation system, you might need to process user-specific events.
Cost Mitigation: To mitigate the cost of network shuffle, consider techniques like:
Hash Partitioning: Use consistent hashing to ensure even distribution.
Aggregations Before Shuffling: Perform aggregations before applying keyBy to reduce the number of partitions and network traffic.
Stateful Processing: Leverage Flink’s state management features to minimize the amount of data shuffled.

#### Closing Summary

KeyBy plays a critical role in stream processing by determining how data is partitioned. In high-cardinality scenarios, it can lead to significant network shuffle costs. Understanding these impacts helps in designing efficient streaming applications that balance consistency requirements with performance optimizations.

#### Common Mistakes
and Follow-Ups
Ignoring Key Cardinality: Developers often assume a low cardinality for keys without considering the potential impact.
Overly Complex Key Functions: Using overly complex key functions can lead to unnecessary shuffles. Simpler, more deterministic key functions are generally preferable.
Performance Tuning: Regular monitoring and profiling are essential to identify performance bottlenecks related to network shuffle.

**Comparison:**

```
tableDiagram
    userId -->|shuffled| partition
    partition -->|processed| result
```

**Code:**

```python
# Example Flink job with KeyBy
from pyflink.dataset import ExecutionEnvironment
from pyflink.table import StreamTableEnvironment, DataTypes
from pyflink.table.descriptors import Schema, OldCsv, FileSystem

env = ExecutionEnvironment.get_execution_environment()
t_env = StreamTableEnvironment.create(env)

t_env.connect(FileSystem().path('input_path')) \
    .with_format(OldCsv()
                 .field('userId', DataTypes.INT())
                 .field('eventType', DataTypes.STRING())) \
    .with_schema(Schema()
                 .field('userId', 'INT')
                 .field('eventType', 'STRING')) \
    .register_table_source('events')

t_env.from_path('events').key_by(lambda x: x.userId) \
    .window(Tumble over '10 minutes' on 'userId' as 'w') \
    .group_by(...) \
    .select(...)
```

**Example input / output:**

```text
Input:
(userId: int, eventType: string)
(1, login)
(2, purchase)
(3, logout)
(4, purchase)
(5, view)

Output (after keyBy(userId)):
Partition 0: [(1, login)]
Partition 1: [(2, purchase)]
Partition 2: [(3, logout)]
Partition 3: [(4, purchase)]
Partition 4: [(5, view)]

Shuffle Input Bytes: 1000 bytes
Shuffle Output Bytes: 800 bytes
```

**Data:**

```json
{
  "userId": [1, 2, 3, 4, 5],
  "eventType": ["login", "purchase", "logout", "purchase", "view"]
}
```

**Flow:**

1. Input Stream
2. KeyBy(userId)
3. Partitioning
4. Network Shuffle
5. Processing

