---
type: "practice"
difficulty: "Expert"
---

# Your cross-AZ replication traffic on a data-ingest Kafka cluster dominates the cloud bill; which levers (fetch-from-follower, compression, retention, RF) would you pull first and why?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Kafka Capacity and Cost](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Kafka_Streams_and_Event_Time/Kafka_Capacity_and_Cost.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with cross-AZ replication traffic in a Kafka cluster that is dominating the cloud bill, it's crucial to understand the trade-offs between various configuration parameters such as fetch-from-follower, compression, retention, and replication factor (RF). By analyzing these levers from first principles, we can determine which adjustments will have the most significant impact on reducing costs while maintaining necessary data integrity and availability.

First, let’s break down what each lever does:

Fetch-from-Follower: Controls how often a leader fetches messages to replicate to followers. Reducing this value means less frequent inter-AZ replication.
Compression: Enforces compression of messages before sending them over the network. This can reduce bandwidth usage significantly but may increase CPU load on both producers and consumers.
Retention: Determines how long messages are retained in Kafka topics. Shorter retention periods mean fewer messages to replicate, which reduces traffic.
Replication Factor (RF): The number of copies of each message that are stored across different brokers. Lowering RF can reduce inter-broker replication but may impact data availability and fault tolerance.
Step-by-Step Analysis

Start with Fetch-from-Follower:

This is often the easiest lever to pull because it directly controls the frequency of inter-AZ replication.
By reducing this value, you significantly cut down on unnecessary traffic without drastically affecting the overall system behavior. For example, setting fetch.min.bytes and fetch.max.bytes can control how often fetches are initiated.

#### Cons
ider Compression:

If your messages are large and frequent, enabling compression can be very effective in reducing bandwidth usage.
However, this will add a bit of latency due to the CPU overhead of compressing and decompressing data on both ends (producer and consumer).

Adjust Retention Time:

Shortening the retention period means fewer messages are replicated across AZs, which directly reduces traffic costs.
Be cautious with very short retention periods as it can affect your ability to query historical data.

Reduce Replication Factor (RF):

Lowering RF can drastically reduce inter-broker replication but comes at the cost of reduced fault tolerance and potential downtime if a broker goes down.
This should be done carefully, considering the business requirements for data availability and consistency.
Practical Example

Let's assume you have a Kafka cluster with 3 replicas per topic (RF=3) in two AZs. If cross-AZ replication traffic is high due to frequent writes or large message sizes, here’s how you might proceed:

Reduce Fetch-from-Follower:

Adjust fetch.min.bytes and fetch.max.bytes to reduce the frequency of fetch requests.
For example, set these parameters lower than their defaults (e.g., 1024B for min bytes, 5MB for max bytes).

Enable Compression:

Set compression.type to lz4 or snappy, depending on your environment and message size.
Monitor CPU usage and latency before fully deploying compression.

Shorten Retention Time:

Reduce the retention time from 7 days to a more reasonable value like 2-5 days, based on business needs for historical data access.
This reduces the amount of data needing replication but might affect your ability to recover older data.

Lower Replication Factor (RF):

If the cost savings are significant enough and the risk of reduced fault tolerance is acceptable, you can lower RF from 3 to 2 or even 1.
Monitor the system closely for any issues like increased message lag or higher failure rates due to reduced redundancy.

#### Closing Summary

In summary, starting with fetch-from-follower adjustments is often the most straightforward and effective way to reduce cross-AZ replication traffic. Compression can be a powerful tool if you have large messages, but it does introduce additional CPU overhead. Shortening retention times offers quick cost savings at the risk of reduced historical data availability. Lowering RF provides significant cost benefits but must be carefully balanced against potential service disruptions.

#### Common Mistakes
and Follow-Ups
Over-aggressively reducing fetch-from-follower: This can lead to increased latencies if not managed properly.
Ignoring CPU implications for compression: Compression can introduce additional overhead, so monitor performance closely.
Failing to consider the business impact of reduced retention: Historical data is often critical and should be weighed against cost savings.

**Comparison:**

```
Topic	Default Settings	Adjusted Settings
RF	3	2
fetch.min.bytes	16384	1024
compression.type	None	lz4
log.retention.hours	2592000	172800
```

**Code:**

```python
# Example configuration adjustments in Kafka properties file
fetch.min.bytes = 1024
fetch.max.bytes = 5242880 # 5MB
compression.type = lz4
log.retention.hours = 72 # 3 days instead of 7
replica.fetch.max.bytes = 67108864 # 64MB (default is 50MB)
```

**Example input / output:**

```text
# Example input and output for adjusting Kafka properties

Input:
- Default settings (RF=3, fetch.min.bytes=16384, compression.type=None, log.retention.hours=2592000)

Output:
- Adjusted settings (RF=2, fetch.min.bytes=1024, compression.type=lz4, log.retention.hours=172800) (2 days)
```

**Data:**

```json
{
  "topics": {
    "topic1": {
      "replication.factor": 2,
      "retention.ms": 345600000 // 4 days in milliseconds
    },
    "topic2": {
      "compression.type": "snappy"
    }
  }
}
```

**Flow:**

1. Check Current Config
2. Fetch-from-Follower
3. Adjust fetch.min.bytes andmax.bytes
4. Monitor Latency andThroughput
5. Consider Compression
6. Enable lz4 or snappyCompression
7. Monitor CPU Usage
8. Reduce Retention Time
9. Adjust log.retention.hours to2-3 days
10. Monitor Historical Data Access
11. Lower Replication Factor (RF)
12. Reduce RF from 3 to 2 or 1
13. Monitor Service Availabilityand Latency

