---
type: "practice"
difficulty: "Expert"
---

# You must standardize on one message broker for the org but teams need queues, streams, and pub-sub. Compare Kafka, Pulsar, SQS/SNS, and NATS as a strategic platform choice, and defend how you'd handle the workloads the winner serves badly.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ API Composition and BFF](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Idempotent_Order_APIs/API_Composition_and_BFF.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To standardize on one message broker for your organization while supporting diverse use cases such as queues, streams, and pub-sub, we need to compare Kafka, Pulsar, SQS/SNS (AWS Simple Queue Service/Simple Notification Service), and NATS. Each has its strengths and weaknesses, making the choice strategic based on specific needs like scalability, performance, ease of integration, and cost.

Firstly, let's break down the requirements:

Queues: A reliable and scalable mechanism for task batching and processing.
Streams: Real-time data processing pipelines with low latency.
Pub-sub: An event-driven architecture where messages can be published without knowing the subscribers.
Triaging the Requirements

Kafka: Known for high throughput, scalability, and fault tolerance. It excels in handling large volumes of data and is widely used for real-time data streaming.

Pulsar: Offers a balanced approach between Kafka's features and ease of use with better performance metrics. It’s designed to be highly available and durable.

SQS/SNS (AWS): Simplifies the integration process but may lack some of the advanced features found in other brokers like Kafka or Pulsar, especially for complex real-time data processing.

NATS: Known for simplicity, low overhead, and ease of use. It excels in simplicity and real-time messaging but lacks some advanced features needed for large-scale systems.

First-Principles Analysis
Queues
Kafka: Can be used as a queue by setting appropriate retention policies and message TTLs.
Pulsar: Has a strong focus on both queues and streams, making it versatile.
SQS/SNS: Primarily designed for queues but also supports topic-based pub-sub.
Streams
Kafka: Industry standard for real-time data streaming with excellent performance.
Pulsar: Also very capable in stream processing due to its architecture.
NATS: Not ideal for complex streams; more suited for simpler, high-frequency messaging needs.
Pub-sub
Kafka: Can be used for pub-sub by setting up topics and consumers appropriately.
Pulsar: Strong support for publish/subscribe patterns with topic-level controls.
SQS/SNS: Built-in support for event-driven architectures and is highly integrated with AWS services.
NATS: Also supports pub-sub but less feature-rich compared to Kafka or Pulsar.
Strategic Platform Choice

Given the requirements, Pulsar emerges as a strong contender. Here’s why:

Balanced Performance and Scalability: Pulsar offers high throughput and low latency while maintaining excellent fault tolerance.
Ease of Integration: It integrates well with various ecosystems, including AWS and Azure.
Advanced Features: Supports both queues and streams seamlessly, making it versatile for diverse workloads.
Community Support and Development: Active development and strong community support make it a robust choice.

However, to cover all bases, we might consider a hybrid approach:

Use Pulsar as the primary message broker due to its comprehensive feature set.
For scenarios where AWS integration is crucial (like SQS/SNS), use both Pulsar and SQS/SNS for redundancy and flexibility.
In cases requiring extremely low overhead and simplicity, NATS can be used.
Handling Workloads Kafka Serves Badly

Kafka has some limitations in terms of ease of operation and cost. Here’s how we can mitigate these issues:

Automation: Use Kubernetes or serverless architectures to manage Kafka clusters, reducing operational complexity.
Cost Management: Implement efficient data retention policies and auto-scaling strategies to control costs.
Monitoring and Logging: Set up comprehensive monitoring to ensure performance and availability.

#### Closing Summary

In conclusion, Pulsar offers a balanced approach that meets the diverse requirements of queues, streams, and pub-sub while being highly scalable and performant. A hybrid strategy can be employed to address specific needs, ensuring flexibility and robustness.

#### Common Mistakes/Follow-Ups

Overlooking the need for advanced features in certain use cases.
Ignoring the integration challenges between different brokers.
Not considering the cost implications of choosing a more complex broker like Kafka.

**Comparison:**

| Broker | Queues | Streams | Pub-sub |
|--------|--------|---------|---------|
| Kafka  | Good   | Good    | Good    |
| Pulsar | Good   | Excellent | Good    |
| SQS/SNS | Good | N/A | Excellent |
| NATS   | Good | Simple | Basic |

**Code:**

```python
# Example code snippet to set up a Pulsar producer and consumer in Python
import pulsar

client = pulsar.Client('pulsar://localhost:6650')
producer = client.create_producer('persistent://public/default/my-topic')
producer.send(b'My message')

consumer = client.subscribe('persistent://public/default/my-topic', 'my-subscription')
message = consumer.receive()
print(message.data())
```

**Example input / output:**

| Input | Output |
|-------|--------|
| `producer.send(b'My message')` | Sent 'My message' to topic `my-topic`. |
| `consumer.receive()` | Received message `'My message'` from subscription `my-subscription`. |

**Data:**

```json
{
  "topic": "persistent://public/default/my-topic",
  "subscription": "my-subscription"
}
```

**Flow:**

1. Queue
2. Kafka
3. Pulsar
4. SQS/SNS
5. Stream
6. Kafka
7. Pulsar
8. Pub-sub
9. Kafka
10. Pulsar
11. SQS/SNS
12. NATS
13. NATS
14. NATS

