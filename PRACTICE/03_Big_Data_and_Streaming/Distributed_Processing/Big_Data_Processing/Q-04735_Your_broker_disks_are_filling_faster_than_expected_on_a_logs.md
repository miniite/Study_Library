---
type: "practice"
difficulty: "Deep"
---

# Your broker disks are filling faster than expected on a logs topic; how do you decide between more retention tuning, compression, tiered storage, or sampling?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Kafka Retention and Storage](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Kafka_Retention_and_Storage.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with rapidly growing disk usage in Kafka topics specifically used for logs, it's crucial to analyze and address the root cause effectively. The primary goal is to ensure efficient data processing while minimizing storage costs. In this scenario, we need to consider whether increasing retention tuning, applying compression, using tiered storage, or implementing sampling would be more effective.

Firstly, let’s reframe the question by understanding what each option means and their implications:

Retention Tuning: This involves setting longer retention periods for topics, allowing more historical data to be stored. While this can provide deeper insights over time, it increases storage costs significantly.

Compression: Applying compression can reduce the size of the data, thereby saving on storage space. However, it may also increase processing overhead due to decompression.

Tiered Storage: This strategy involves moving older or less frequently accessed data to cheaper storage solutions, such as cold storage services. It can be highly effective in reducing costs but might introduce complexity and latency for real-time processing.

Sampling: Reducing the volume of data by sampling can significantly decrease the amount of data stored. However, this approach may lead to loss of critical information.

Now, let’s break down each option from first principles:

Retention Tuning:

#### Pros
Retention tuning allows you to capture and analyze historical data for extended periods.

#### Cons
This can be very expensive due to the high storage costs, especially if the logs are generated frequently.

Compression:

#### Pros
Compression reduces the volume of stored data, thereby saving on storage costs without significantly affecting real-time processing.

#### Cons
Compressed data requires additional processing power for decompression, which can introduce latency and overhead.

Tiered Storage:

#### Pros
Tiered storage is highly cost-effective as it offloads older or less frequently accessed logs to cheaper storage solutions.

#### Cons
It introduces complexity in managing different tiers of storage and may impact performance if the cold storage service has slower access times.

Sampling:

#### Pros
Sampling can significantly reduce data volume, making storage more manageable without significant loss of important information.

#### Cons
Depending on the sampling strategy, it might lead to missing critical insights or events, which could be detrimental for certain use cases.
Step-by-Step Solution:

Assess Current Retention Policy:

Determine how much data is currently being stored and at what rate.
Calculate the cost of extending retention periods based on current storage costs.

Evaluate Data Volume and Distribution:

Analyze the distribution of log entries over time to identify patterns in access frequency.
Use this information to decide if tiered storage or compression would be more beneficial.

Apply Compression Strategy:

Implement a compression algorithm suitable for your data type (e.g., Snappy, LZ4).
Measure the impact on storage and processing performance.

Implement Tiered Storage:

Configure Kafka to automatically move logs to cold storage after a certain period.
Ensure that access patterns can handle latency introduced by cold storage services.

#### Cons
ider Sampling Techniques:

Implement sampling strategies such as log-level sampling or timestamp-based sampling.
Evaluate the trade-offs between data accuracy and storage efficiency.

Monitor and Optimize:

Continuously monitor storage usage and performance.
Adjust configurations based on actual usage patterns and cost-benefit analysis.

#### Closing Summary

In summary, the choice between retention tuning, compression, tiered storage, or sampling depends heavily on the specific use case and cost considerations. Retention tuning offers deep historical insights but is expensive; compression saves space with some overhead; tiered storage balances cost and performance; and sampling reduces volume but risks losing critical data.

#### Common Mistakes/Follow-Ups
Misalignment of Goals: Ensuring that chosen strategies align with business goals, such as real-time processing or long-term analytics.
Overlooking Performance Impact: Failing to consider the impact on processing speed when implementing compression or sampling.
Ignoring Cost-Benefit Analysis: Making decisions without thorough cost analysis can lead to suboptimal solutions.

**Comparison:**

```
Option	Pros	Cons
Retention Tuning	Captures historical data for insights.	High storage costs, not cost-effective.
Compression	Reduces storage size with minimal overhead.	Increased processing latency during decompression.
Tiered Storage	Cost-effective and reduces real-time impact.	Additional complexity in management.
Sampling	Reduces volume significantly at lower cost.	May lose critical information.
```

**Code:**

```python
# Example Python code for applying compression in Kafka producer
from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers='localhost:9092', value_serializer=lambda v: zstd.compress(v.encode()))

for line in lines:
    # Compress and send the log data
    producer.send('logs_topic', key=None, value=line)
```

**Example input / output:**

```text
Input:
- Current logs topic with high retention and growing storage costs.

Output:
- Decision to implement compression, move low-access logs to cold storage, and apply sampling during off-peak hours.
```

**Data:**

```json
{
  "topic": "logs_topic",
  "compression_level": 3,
  "retention_ms": 86400000, // 24 hours in milliseconds
  "log_distribution": {
    "peak_hours": [9, 17],
    "access_frequency": {
      "high": ["Monday", "Tuesday"],
      "low": ["Friday", "Saturday"]
    }
  }
}
```

**Flow:**

1. Assess Current RetentionPolicy
2. Evaluate Data Volume
3. Apply Compression Strategy
4. Evaluate Tiered StorageConfiguration
5. Consider Sampling Techniques
6. Monitor and Optimize
7. Reassess based onperformance and cost

