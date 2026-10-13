---
type: "practice"
difficulty: "Expert"
---

# Design multi-region streaming for a payments ledger with an RPO near zero and an RTO under two minutes. How do you handle cross-region Kafka replication, consumer offset consistency, and split-brain during a region partition?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Provisional and Final Results](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a multi-region streaming architecture for a payments ledger with near-zero RPO (Recovery Point Objective) and an RTO (Recovery Time Objective) under two minutes, we need to focus on several key aspects: cross-region Kafka replication, consumer offset consistency, and split-brain handling during region partitioning. The goal is to ensure data integrity, availability, and fault tolerance.

#### Triage

The core of the question revolves around designing a robust, real-time streaming system that can handle distributed failures gracefully. We need to address how to replicate data across regions efficiently, manage consumer offsets consistently, and prevent split-brain scenarios during partitioning events.

#### Assumptions
The ledger requires high availability with minimal downtime.
There are at least two regions (e.g., primary and secondary) for redundancy.
Kafka is chosen as the messaging system due to its high throughput and fault tolerance capabilities.

#### First Principles Solution

Cross-Region Replication:

Utilize Kafka's built-in multi-region capabilities, such as Kafka Connect with S3 or Cassandra for durable storage.
Implement a replication factor of 2 (primary region + secondary region) to ensure at least one copy is available in each region.

#### Cons
umer Offset Consistency:

Use a global leader election mechanism like ZooKeeper to manage leaders and ensure consistent offset handling across regions.
Leverage Kafka's group management features to maintain coherent consumer groups that can be split or merged based on regional availability.

Split-Brain Handling:

Implement a region-aware partitioning strategy using a load balancer that routes messages only to available regions.
Use a distributed coordination service (e.g., Kubernetes) to dynamically manage and reroute traffic in case of failures.
Detailed Implementation
1. Cross-Region Replication with Kafka Connect
Yes
No
Primary Region
Is Primary Region Available?
Replicate Data to SecondaryRegion
Failover to Secondary Region
Store Data in S3/Cassandra
Use Secondary Region asPrimary
2. Consumer Offset Management
Use a distributed offset management system that can be queried across regions.
Implement a custom consumer group leader election mechanism using ZooKeeper or another coordination service.
Yes
No

#### Cons
umer Group
Is Leader Available?
Assign Offset to Consumer
Election for New Leader
Process Messages
Wait for New Leader to TakeOver
3. Split-Brain Handling
Use a service mesh like Istio or Envoy to route traffic based on regional availability.
Implement a circuit breaker pattern to ensure that requests are not overloaded during partitioning events.
Yes
No
Region A
Is Region A Available?
Route Traffic Normally
Fallback to Region B or S3

#### Closing Summary
and Common Mistakes/Follow-Ups

To summarize, the key is to ensure that data replication, offset management, and traffic routing are robust and failover mechanisms are well-defined. The primary challenges include handling split-brain scenarios gracefully and ensuring consistent state across regions.

#### Common mistakes
might include overlooking regional availability during deployment or not implementing a proper fallback mechanism for read operations. Follow-ups could involve discussing specific implementation details of Kafka Connect, ZooKeeper usage, or the architecture of the load balancer.

**Comparison:**

```
Region	Availability	Leader Status
A	Available	True
B	Unavailable	False
```

**Code:**

```python
# Example code snippet for consumer offset management using Apache Kafka and ZooKeeper

from kafka import KafkaConsumer, TopicPartition
import requests

def assign_offset(consumer_group):
    # Poll ZooKeeper to determine leader and assign offsets
    zk_url = "http://zookeeper-service:2181"
    response = requests.get(f"{zk_url}/leaders/{consumer_group}")
    if response.status_code == 200:
        leader = response.json()["leader"]
        for tp in consumer_group.partitions.values():
            offset = assign_offset_to_partition(leader, tp)
            consumer.seek(TopicPartition(tp.topic, tp.partition), offset)

def assign_offset_to_partition(leader, partition):
    # Query ZooKeeper for the latest offset for the given partition
    zk_url = "http://zookeeper-service:2181"
    response = requests.get(f"{zk_url}/offsets/{partition}")
    return int(response.text.strip())
```

**Example input / output:**

```text
Request:
POST /replicate-data HTTP/1.1
Content-Type: application/json

{
  "topic": "payments-ledger",
  "partition": 2,
  "offset": 42,
  "region": "secondary"
}

Response:
HTTP/1.1 200 OK
Content-Length: 0

# Data is replicated to the secondary region with offset 42 for partition 2.
```

**Data:**

```yaml
# Example YAML configuration for Kafka Connect to S3
connect-standalone.properties:
  bootstrap.servers=kafka-cluster:9092
  key.converter=org.apache.kafka.connect.storage.StringConverter
  value.converter=org.apache.kafka.connect.storage.StringConverter

kafka-connect-s3:
  tasks.max: 1
  connector.class: io.confluent.connect.s3.S3SinkConnector
  topics: payments-ledger
  s3.region: us-west-2
  flush.size: 10485760 # 10MB
```

**Flow:**

1. Yes
2. No
3. Region A
4. Is Region A Available?
5. Route Traffic Normally
6. Fallback to Region B or S3

