---
type: "practice"
difficulty: "Deep"
---

# How would you design topics and keys for a multi-tenant analytics platform so one big tenant does not create a hot partition?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Partition Count Decisions](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Partition_Count_Decisions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: Designing Kafka topics and keys for a multi-tenant analytics platform involves ensuring data distribution is even across tenants to avoid hot partitions, particularly when dealing with a single large tenant. This requires careful consideration of key generation strategies, topic partitioning, and data modeling techniques.

To start, let's clarify the question: The core challenge here is preventing a single large tenant from overwhelming one or more Kafka partitions, leading to a "hot partition." We need to ensure that data for all tenants is evenly distributed across available Kafka partitions. This involves understanding how keys are generated, partitioning strategies, and possibly implementing custom logic to handle uneven tenant sizes.

#### Assumptions
include:

The platform processes high volumes of data.
Tenants can vary significantly in size and activity.
We want to minimize the risk of a single large tenant affecting other tenants' performance.

Step-by-step, we'll design our Kafka topics and keys:

Identify Unique Keys: For each message, use a key that uniquely identifies the tenant. This ensures messages from different tenants are distributed across partitions.

Partitioning Strategy: Use a partitioner that distributes data based on the key in a deterministic manner. In Kafka, this is usually done via the partitionFor method provided by the Kafka producer client. For our case, we can use a hash-based partitioner which ensures that messages with similar keys are distributed evenly.

Custom Partitioning Logic: If the above is not sufficient to distribute data evenly, implement custom logic within the Kafka producer or consumer to ensure even distribution. This could involve:

Using a hashing function on the tenant ID and other relevant fields.
Implementing a dynamic partitioner that can adjust based on real-time data distribution.

Topic Creation: Create topics with an appropriate number of partitions (e.g., 30-100) to ensure even load distribution.

#### Cons
umer Group Management: Ensure that consumer groups are set up to handle the load properly, possibly by rebalancing strategies and group offsets management.

Monitoring and Alerting: Set up monitoring for Kafka topics to detect any signs of hot partitions. Use tools like Prometheus and Grafana to visualize this data.

Let's go through a step-by-step example:

Step 1: Key Generation
PYTHON
def generate_key(tenant_id, message):
    # Generate a unique key based on tenant ID and other relevant fields
    return f"{tenant_id}:{hash(message)}"
Step 2: Custom Partitioner Implementation
PYTHON
class CustomPartitioner:
    def partition(self, topic_name, key_bytes, num_partitions):
        # Custom logic to hash the key and map it to a partition
        tenant_id = key_bytes.split(b':')[0]
        hash_value = abs(hash(tenant_id)) % num_partitions
        return int(hash_value)
Step 3: Topic Creation in Kafka Producer
PYTHON
from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers='localhost:9092',
                         key_serializer=lambda k: k.encode('utf-8'),
                         value_serializer=lambda v: json.dumps(v).encode('utf-8'))

for message in messages:
    tenant_id = message['tenantId']
    key = generate_key(tenant_id, message)
    producer.send(topic='analytics', key=key, value=message)

producer.flush()
Step 4: Consumer Group Management
PYTHON
from kafka import KafkaConsumer

#### cons
umer = KafkaConsumer('analytics',
                         bootstrap_servers='localhost:9092',
                         group_id='tenant_analytics_group',
                         auto_offset_reset='earliest')

for message in consumer:
    print(f"Received message from {message.key.decode()}: {message.value}")

#### Closing Summary

By following the above steps, we can ensure that a large tenant does not create a hot partition. The key is to use unique keys for each message, apply custom partitioning logic if needed, and monitor the system to maintain even load distribution.

#### Common Mistakes

Using non-unique or poorly distributed keys.
Not considering dynamic rebalancing of consumer groups.
Ignoring monitoring and alerting for early detection of hot partitions.

