---
type: "practice"
difficulty: "Deep"
---

# What is the Kappa architecture and how is it different from the Lambda architecture?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Streaming Engines and Kafka Consumers](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Streaming_Engines_and_Kafka_Consumers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The Kappa architecture, also known as the Event-Driven Data Pipeline, represents a unified approach to handling both batch and stream processing in a single system. It contrasts with the traditional Lambda architecture, which processes data through two separate paths: one for real-time streaming and another for batch processing. In this overview, we will explore the Kappa architecture's principles, its benefits over the Lambda architecture, and discuss key differences.

Firstly, let’s restate what the question is probing. The interviewer wants to understand your grasp of both architectures and how they differ in terms of design philosophy, scalability, and operational complexity. You should make assumptions about the context, such as whether we are dealing with large-scale data processing environments or more common scenarios. We will also reason step-by-step from first principles to fully explain why Kappa is a more modern approach.

The Kappa architecture combines batch and stream processing into a single framework. Instead of having separate pipelines for real-time and historical data, it processes all incoming events in the same pipeline but uses different computation methods based on whether they are recent or not. This unified design simplifies system complexity and reduces costs by eliminating redundant components.

To understand the Kappa architecture better, let’s break down its core components:

Event Source: Data is ingested from various sources (e.g., sensors, logs).
Event Broker: A layer that stores events as they are received. These events can be stored in a fault-tolerant manner like Kafka or Apache Pulsar.
Processing Layer: This layer applies real-time and batch processing logic to the data. Real-time queries use micro-batch processing with sliding windows, while historical analysis uses traditional batch processing techniques.

In contrast, the Lambda architecture maintains two separate pipelines:

Real-Time Path (Lambda): Handles real-time updates using stream processing frameworks like Apache Storm or Flink.
Batch Path: Processes historical data to generate reports and summaries.
Step-by-Step Explanation

Event Ingestion:

Both architectures start with the same event ingestion process, where raw data is collected from various sources such as IoT devices, web servers, etc.

Event Storage:

Kappa architecture uses a single layer of storage to hold both real-time and historical events.
Lambda stores these events separately, leading to redundant infrastructure costs.

Processing Logic:

In Kappa, the same processing logic is applied across all events regardless of their freshness. The system differentiates by applying micro-batching techniques for real-time queries.
For batch processing in Lambda, data is first stored and then processed periodically using Hadoop or Spark.

Cost and Complexity:

Kappa simplifies the infrastructure as it uses a single storage layer, reducing costs and complexity.
Lambda requires maintaining two separate pipelines, leading to higher operational overhead.

Scalability and Fault Tolerance:

Both architectures can scale horizontally by adding more nodes to handle increased load.
However, Kappa's unified approach often results in better fault tolerance due to its single storage layer design.

#### Closing Summary

In summary, the Kappa architecture offers a unified solution that processes both real-time and historical data through a single pipeline. This reduces redundancy and operational complexity compared to the Lambda architecture. The key differences lie in how they handle event ingestion, storage, processing logic, cost, and scalability.

#### Common Mistakes/Follow-Ups
Forgetting to mention the importance of micro-batch processing in Kappa for real-time queries.
Not discussing potential trade-offs related to data freshness or latency.
Overlooking the need for robust event sourcing practices to ensure consistency and reliability.

**Comparison:**

```
Architecture	Kappa	Lambda
Storage	Single layer (Kafka)	Separate layers (Kafka + HDFS)
Processing	Micro-batch and batch processing	Stream and batch processing separately
Complexity	Lower due to unified design	Higher with two separate pipelines
```

**Code:**

```python
# Example code snippet for ingesting events into a Kafka topic
from kafka import KafkaProducer

def send_event_to_kafka(event):
    producer = KafkaProducer(bootstrap_servers='localhost:9092')
    future = producer.send('events_topic', b'raw event data')
    result = future.get(timeout=10)
```

**Example input / output:**

```text
Input: Event sent to Kafka topic 'events_topic'
Output: Event successfully stored in the event broker for processing.
```

**Data:**

```json
{
  "event": {
    "timestamp": "2023-10-05T14:48:00Z",
    "source": "sensor_123",
    "value": 75.2,
    "type": "temperature"
  }
}
```

**Flow:**

1. Yes
2. No
3. Event Source
4. Kafka Broker
5. Is event recent?
6. Micro-batch processing usingFlink
7. Batch processing usingSpark/Hadoop

