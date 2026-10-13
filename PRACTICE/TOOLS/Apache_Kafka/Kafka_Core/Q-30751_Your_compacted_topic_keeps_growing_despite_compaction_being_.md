---
type: "practice"
difficulty: "Deep"
---

# Your compacted topic keeps growing despite compaction being enabled. What operational reasons prevent compaction from reclaiming space and how do you fix each?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Log Compaction](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Log_Compaction.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Compacted Kafka topics are designed to keep only the most recent message, but sometimes they don't shrink as expected due to several operational reasons. This scenario can be tricky because even when compaction is enabled, various factors might interfere with its functionality. Let's break down these issues and how to address them.

#### Triage

The question asks about why a compacted Kafka topic is not reclaiming space despite having compaction enabled. We need to consider several operational aspects such as retention policies, producer behavior, consumer offsets, and Kafka cluster health. The interviewer might be probing for an understanding of the underlying mechanics of Kafka's compaction mechanism, the importance of configuration settings, and the ability to diagnose and fix issues.

#### First-Principles Solution
Retention Policy: Ensure that the retention policy is correctly configured. Compaction works only within a topic’s retention period.
Producer Behavior: Verify how producers are sending messages. If producers frequently send duplicates or overlapping messages, it can prevent compaction from running effectively.

#### Cons
umer Offsets: Check if consumers are consuming messages in such a way that they might be holding onto offsets for too long.
Cluster Health and Load: Evaluate the overall health of the Kafka cluster to ensure there's sufficient resources available.
Detailed Steps

Check Retention Policies

Verify the retention.ms and segment.bytes settings in your topic configuration. Ensure that messages are being retained for a period long enough to allow compaction.
BASH
kafka-topics.sh --describe --topic <your_topic> --zookeeper localhost:2181
If retention is set too low, increase it and observe if compaction improves.

Examine Producer Behavior

Review the producer configuration to ensure no duplicates or overlapping messages are being sent.
Use tools like kafka-console-consumer.sh to inspect message payloads and identify any anomalies.
BASH
kafka-console-consumer.sh --bootstrap-server <your_kafka_broker> --topic <your_topic> --from-beginning

Analyze Consumer Offsets

Use Kafka’s offset management tools to ensure consumers are not holding offsets for too long, which can interfere with compaction.
BASH
kafka-consumer-groups.sh --describe --group <your_consumer_group> --bootstrap-server <your_kafka_broker>

Monitor Cluster Health and Load

Use tools like kafkacat, kafka-producer-perf-test, or Kafka Manager to monitor the cluster's overall health.
BASH
kafkacat -C -b <your_kafka_broker> -L

#### Closing Summary
and Common Mistakes

To ensure compaction works effectively, it’s crucial to maintain proper retention policies, manage producer behavior, monitor consumer offsets, and keep the Kafka cluster healthy. Common mistakes include misconfiguring retention periods, sending redundant messages, and ignoring cluster health issues.

#### Follow-Ups
How do you ensure data integrity while performing compaction?
What are some strategies for optimizing message deduplication in producers?
Can you discuss the trade-offs between compaction frequency and resource usage?

**Comparison:**

```
Kafka Topic Configuration	Value
retention.ms	604800000 (7 days)
segment.bytes	1073741824 (1 GB)
cleanup.policy	compact,delete
```

**Code:**

```bash
# Example command to check topic retention policy
kafka-topics.sh --describe --topic <your_topic> --zookeeper localhost:2181

# Example command to inspect messages from a consumer group
kafka-console-consumer.sh --bootstrap-server <your_kafka_broker> --topic <your_topic> --from-beginning
```

**Example input / output:**

```text
# Example output of topic description
Topic: my_topic
PartitionCount: 5
ReplicationFactor: 1
Config:
    retention.ms = 604800000
    segment.bytes = 1073741824

# Example command to check consumer group offsets
kafka-consumer-groups.sh --describe --group my_group --bootstrap-server localhost:9092
```

**Data:**

```yaml
# Sample topic configuration
retention.ms: 604800000 # 7 days in milliseconds
segment.bytes: 1073741824 # 1 GB in bytes
cleanup.policy: compact,delete
```

**Flow:**

1. Check Retention Policy
2. Verify `retention.ms` and`segment.bytes`
3. Run 'kafka-topics.sh--describe' to inspect topicconfig
4. Set retention policy ifnecessary
5. Examine Producer Behavior
6. Review producer configurationfor duplicates
7. Inspect message payloads with`kafka-console-consumer.sh`
8. Adjust producer settings ifneeded
9. Analyze Consumer Offsets
10. Use'kafka-consumer-groups.sh--describe' to check offsets
11. Ensure consumers are notholding onto outdated offsets
12. Monitor Cluster Health andLoad
13. Use tools like kafkacat or KafkaManager
14. Identify and address clusterhealth issues

