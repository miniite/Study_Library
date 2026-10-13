---
type: "practice"
difficulty: "Deep"
---

# Your metrics producers batch aggressively but a critical health-check event sits waiting for the batch to fill. How do you serve both bulk telemetry and urgent signals through one producer?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Pipeline Lag Monitoring](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Pipeline_Lag_Monitoring.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need to design a Kafka producer that can handle both bulk telemetry and critical health-check events efficiently. The core idea is to ensure that important signals like health checks are not delayed due to aggressive batching of telemetry data. Here’s how we can achieve this:

Understand the Problem: We have a metrics producer configured for high volume, batched telemetry data (e.g., collected every few seconds). However, there's a critical need to ensure that urgent signals like health-check events are delivered immediately and reliably.

Key Assumptions:

The health-check event is time-critical and must be processed quickly.
Telemetry data can be batched for efficiency but should not delay the processing of urgent events.
Kafka is used as the messaging system, with producers that can control batching and delivery semantics.

#### First Principles Solution

Implement a producer strategy that prioritizes critical health-check events while still allowing bulk telemetry data to be sent in batches.
Use separate topic or partition strategies for different event types (telemetry vs. health checks) if necessary.
Leverage Kafka's delivery guarantees and batching capabilities judiciously.

Detailed Solution:

Define Event Types: Identify distinct message types: telemetry data, and critical health-check events. Both can be sent through the same producer but with different handling logic.
Batching for Telemetry Data: Configure the Kafka producer to batch telemetry data aggressively (e.g., every 10 seconds). Use settings like batch.size and linger.ms to control how long the producer waits before sending a message even if it has not reached the configured size. This is crucial for efficient bulk data transmission.
Priority Handling: For critical health-check events, ensure they are given priority over telemetry messages. One approach could be using a custom serialization strategy where each message type is tagged with a priority level (e.g., 1 for telemetry, 2 for health checks), and the producer logic ensures that higher-priority messages are sent before lower-priority ones.
Topic Partitioning: Consider having separate partitions or topics for different event types. For example, one partition/topic can handle health-check events exclusively, while another handles telemetry data. This helps in distributing load and ensuring critical messages get priority.
Delivery Semantics: Use at-least-once delivery semantics to ensure that no important message is lost but allow the flexibility of retrying if needed. For telemetry data, this might be acceptable since it can tolerate some latency.

#### Closing Summary
By implementing a producer strategy that distinguishes between different event types and uses appropriate Kafka configurations, we can achieve both efficient bulk telemetry transmission and immediate handling of critical health-check events. The key is to balance the batching requirements for telemetry with the urgency of delivering important signals like health checks.

#### Common Mistakes
and Follow-Ups:

A common mistake might be treating all messages equally without considering their importance, leading to delayed delivery of critical information.
Another issue could arise from overly aggressive batch sizes for telemetry data, potentially delaying important events.
Interviewers might ask about performance implications, how to handle failure scenarios, and whether there are alternative messaging systems or patterns that could be more suitable.

**Comparison:**

```
Event Type	Priority	Sending Strategy
Telemetry Data	1	Batched, delayed sending
Health Check Event	2	Immediate delivery
```

**Code:**

```java
// Example Java Kafka producer configuration with custom serialization logic

Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("acks", "all"); // Ensure at-least-once delivery
props.put("batch.size", 16384); // Batching for telemetry data
props.put("linger.ms", 10);     // Wait up to 10ms before sending a batch

KafkaProducer<String, String> producer = new KafkaProducer<>(props);

// Custom serialization based on message type and priority
public void send(String topic, String key, String value, int priority) {
    if (priority >= 2) { // High-priority health check event
        producer.send(new ProducerRecord<>(topic, key, value));
    } else { // Telemetry data
        // Use custom logic to batch and delay sending of telemetry events
    }
}
```

**Example input / output:**

```text
Input:
- Telemetry event at 2023-10-01T14:00:05Z with value 5.6 and priority 1.
- Health check event at 2023-10-01T14:01:00Z with status "OK" and priority 2.

Output:
Health check event sent immediately.
Telemetry event batched for transmission after a short delay or on reaching the configured size/batch limit.
```

**Data:**

```json
{
  "telemetryEvent": {
    "timestamp": "2023-10-01T14:00:00Z",
    "value": 5.6,
    "priority": 1
  },
  "healthCheckEvent": {
    "timestamp": "2023-10-01T14:01:00Z",
    "status": "OK",
    "priority": 2
  }
}
```

**Flow:**

1. Yes
2. No
3. Receive Event
4. Is High Priority?
5. Send Immediately
6. Batch with Other TelemetryData
7. Sent to Kafka
8. Wait for Batch Size or Timeout

