---
type: "practice"
difficulty: "Intermediate"
---

# A consumer is lagging behind on a busy telemetry topic; what basic steps would you take to find out why?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Consumer Lag Diagnosis](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Consumer_Lag_Diagnosis.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To diagnose and address a lagging Kafka consumer in a busy telemetry topic, we need to systematically gather information about the consumer's behavior and the underlying system. The key is to start with basic steps that can help us pinpoint the issue without overcomplicating things.

#### Triage
What’s Really Being Probed

The interviewer is likely probing your ability to perform root cause analysis in a distributed system, understanding Kafka consumer mechanisms, and troubleshooting data processing pipelines. Key assumptions include:

The consumer is part of a larger data pipeline involving Spark or another big data framework.
There are no immediate hardware failures (e.g., disk space issues).
The topic has sufficient throughput that the consumer can't keep up.
Step-by-Step Solution

Check Consumer Offsets:

Verify if the consumer has fallen behind by examining its offsets. Use kafka-consumer-groups.sh tool to check the lag.
BASH
./kafka-consumer-groups.sh --bootstrap-server <broker-url> --describe --group <consumer-group-id>
If the offsets are significantly behind, proceed with further diagnostics.

Review Consumer Logs:

Check the consumer logs for any errors or warnings that could indicate issues like deserialization failures.
BASH
tail -f /path/to/consumer/logs/*.log

Examine Kafka Topic Partitioning and Replication:

Ensure the topic is properly partitioned to handle the load. Use kafka-topics.sh to check the partition details:
BASH
./kafka-topics.sh --bootstrap-server <broker-url> --describe --topic <topic-name>
Verify that there are enough partitions and replicas to avoid single points of failure.

Monitor Network Latency:

Check network latency between brokers and consumers using tools like ping or traceroute.

Spark Application Metrics:

If the consumer is part of a Spark application, review the Spark UI for task failures, shuffle operations, and other metrics that could indicate bottlenecks.
BASH
http://<spark-master-url>:4040

Resource Utilization:

Monitor CPU, memory, and disk I/O usage on both the consumer nodes and Kafka brokers to ensure there are no resource constraints.

#### Cons
umer Configuration:

Check if the consumer configuration is optimal for the current load. Key configurations include fetch.min.bytes, auto.offset.reset, and enable.auto.commit.

Topic Retention Policies:

Review topic retention settings in Kafka to ensure they are appropriate for your data volume and processing needs.

#### Closing Summary

By following these steps, we can systematically narrow down the root cause of a lagging consumer and take corrective actions. The key is to methodically gather information about both the consumer’s state and the surrounding environment before diving into complex diagnostics.

#### Common Mistakes/Follow-Ups
Ignoring Logs: Overlooking log files can hide critical information.
Jumping Too Quickly to Code Fixes: Addressing a consumer lag typically involves adjusting configurations or monitoring rather than immediate coding.
Focusing Solely on CPU Metrics: While CPU is important, it’s equally crucial to consider memory and disk I/O as well.

**Comparison:**

```
Partition	Replicas	Lag
0	1,2	753K
1	3,4	N/A
```

**Code:**

```bash
# Example command for Kafka consumer group description
./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-group

# Example tailing logs
tail -f /path/to/consumer/logs/*.log

# Example Spark UI URL
http://localhost:4040/
```

**Example input / output:**

```text
# Example output from kafka-consumer-groups.sh
GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-DLL              HOST      CLIENT-ID
my-group        telemetry-topic 0          1234567         9876543         7530876         /path/to/consumer.jar     localhost  my-consumer

# Example log output
[2023-11-10 14:30:00] INFO [Consumer clientId=consumer-1, groupId=my-group]: Received and acknowledged offsets for partition telemetry-topic-0 in 50ms.
[2023-11-10 14:31:00] WARN [Consumer clientId=consumer-1, groupId=my-group]: Failed to fetch metadata with correlation id 123 from broker 1.92.168.1.1:9092 (localhost/127.0.0.1:45116) due to Timeout when fetching metadata for topics [telemetry-topic]
```

**Data:**

```json
{
  "topic": {
    "name": "telemetry-topic",
    "partitions": [
      {"id": 0, "replicas": [1, 2]},
      {"id": 1, "replicas": [3, 4]}
    ]
  }
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. No
5. Check Offsets
6. Are offsets behind?
7. Examine logs
8. Are there errors?
9. Adjust consumer config
10. Monitor network latency
11. Verify topic partitioning andreplication

