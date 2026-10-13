---
type: "practice"
difficulty: "Deep"
---

# You run a 300-partition telemetry topic and a rebalance storm keeps stalling ingestion during deploys; how would you diagnose and stabilize the consumer group?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Consumer Lag Diagnosis](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Consumer_Lag_Diagnosis.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this issue, let's first outline the core problem: a Kafka consumer group responsible for ingesting data from a 300-partition topic is experiencing stalls during deployment, due to an ongoing rebalance storm. The key challenge here involves diagnosing and stabilizing the consumer group to ensure smooth operation.

#### Triage

The question is asking us to diagnose why the ingestion of telemetry data is stalling despite frequent rebalances in a Kafka cluster with 300 partitions. The interviewer is probing our understanding of Kafka's partitioning, rebalance mechanics, resource constraints, and potential bottlenecks affecting consumer performance during deployment.

Diagnosing the Issue
Step-by-Step Analysis from First Principles

Understand Partition Distribution:

Each topic in Kafka has a set number of partitions.
In this case, the telemetry topic has 300 partitions, which means there are 300 consumer group instances handling data.

Rebalance Mechanics:

When consumers join or leave a consumer group, rebalances occur to reassign partitions among the live consumers in the group.
Frequent rebalances can cause delays if the system is underutilized or resources are insufficiently allocated.

Resource Constraints:

Insufficient CPU, memory, or network bandwidth on brokers can slow down consumer processing and lead to stalls during rebalances.

#### Cons
umer Lag:

Excessive lag between partitions could indicate that consumers are not keeping up with the producer rate.
This is a common symptom of resource constraints or inefficient consumer logic.

Network Latency:

High network latency can cause delays in message delivery, affecting the timely processing of messages by consumers.

#### Solution
Approach

Monitor Consumer Metrics:

Use tools like Kafka’s built-in metrics, Prometheus, or Grafana to monitor partition lag, consumer offset, and throughput.
Check the consumer group’s lag using kafka-consumer-groups.sh script.

Adjust Partition Assignments:

Evaluate if the current number of partitions is optimal for your workload.

#### Cons
ider increasing the number of consumers in the consumer group to handle more partitions concurrently.

Optimize Consumer Logic:

Ensure that the consumer logic is optimized and can process messages efficiently without causing backpressure.
Avoid blocking operations within the consumer’s processing pipeline.

Resource Allocation:

Verify that each broker has adequate CPU, memory, and network resources to handle its partitions.

#### Cons
ider adding more brokers or optimizing existing ones if resource constraints are a factor.

Tuning Kafka Configuration:

Adjust settings like max.partition.fetch.bytes and fetch.min.bytes to control the amount of data fetched in each request.
Tune session.timeout.ms and rebalance.backoff.ms to manage rebalance intervals more effectively.

#### Closing Summary

In summary, diagnosing and stabilizing a consumer group during deployment involves a combination of monitoring metrics, optimizing resource allocation, tuning Kafka configurations, and refining the consumer logic. The key is to identify bottlenecks, address them systematically, and ensure that all components are well-coordinated for smooth operation.

#### Common mistakes
include overlooking resource constraints, failing to monitor lag effectively, and not adjusting partition assignments or consumer settings based on workload analysis.

**Comparison:**

```
Partition	Leader Broker ID	Replicas	ISR
0	1	[1]	[1]
...	...	...	...
299	3	[3,4]	[3]
Partition	Lag (ms)
0	5000
...	...
299	8000
```

**Code:**

```python
# Example code snippet for monitoring Kafka metrics using Python
from kafka import KafkaConsumer

consumer = KafkaConsumer('telemetry-topic', bootstrap_servers=['localhost:9092'])
consumer.poll()
```

**Example input / output:**

```text
Input:
kafka-consumer-groups.sh --describe --group telemetry-consumer-group

Output:
Topic           Partition       Leader    Replicas     Isr
telemetry-topic      0              1         [1]        [1]
...
telemetry-topic     299             3         [3,4]      [3]

Partition Lag for Partitions in Group telemetry-consumer-group:
Partition: 0 - Lag: 5000 ms
Partition: 1 - Lag: 7000 ms
...
Partition: 299 - Lag: 8000 ms
```

**Data:**

```json
{
  "brokers": [
    {"id": 1, "cpu": 4, "memory": 32},
    {"id": 2, "cpu": 6, "memory": 64}
  ],
  "partitions": [
    {"topic": "telemetry-topic", "partition": i} for i in range(1, 301)
  ]
}
```

**Flow:**

1. Monitor Consumer Metrics
2. Check Partition Lag
3. Tune Kafka Configuration
4. Adjust Partition Assignments
5. Optimize Consumer Logic
6. Verify Resource Allocation
7. Rebalance and MonitorContinuously

