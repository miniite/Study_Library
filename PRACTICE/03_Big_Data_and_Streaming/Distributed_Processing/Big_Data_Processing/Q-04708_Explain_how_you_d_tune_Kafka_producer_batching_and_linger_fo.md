---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd tune Kafka producer batching and linger for a high-throughput agent fleet without hurting alert latency on the critical path.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Consumer Lag Diagnosis](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Consumer_Lag_Diagnosis.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to fine-tune Kafka producers' batching and linger settings to ensure our AIOps system can handle high throughput from a large number of agents while maintaining low alert latency in critical paths. The key balance here is between maximizing message throughput and minimizing delays that could impact real-time alerts.

#### Triaging the Question

The core issue involves optimizing producer configurations for Kafka. Specifically, we need to adjust batch.size (the size of the batch) and linger.ms (how long to wait before sending a batch if not full). We must ensure these settings do not introduce unacceptable delays in alert processing.

#### Assumptions
The agents are generating a large volume of low-latency data.
Critical alerts need to be processed within strict time constraints.
We want to minimize the risk of increasing latency across the board while maximizing throughput.
Step-by-Step Solution
1. Understanding Kafka Producer Settings

Kafka producer settings like batch.size and linger.ms are crucial for tuning performance:

batch.size: The size of each batch in bytes. A larger batch means more data can be sent in one go, reducing the number of network round trips but potentially increasing latency if not filled.
linger.ms: This controls how long a producer waits to accumulate messages before sending them out even if the batch is not full. Setting this too low might result in unnecessary delays; setting it too high could increase throughput at the cost of higher latency.
2. Tuning batch.size

Given that we need both high throughput and minimal alert delay, we should aim for a balance where the batches are large enough to reduce the number of network requests but small enough to avoid excessive accumulation times. A typical starting point might be around 16 KB, which is often a good balance between throughput and latency.

3. Tuning linger.ms

linger.ms settings should be adjusted based on the criticality of the data being sent. For non-critical data, this can be set higher to batch more messages together; for critical alerts, it should be lower to ensure timely delivery.

Let's assume a scenario where we have 100 agents sending logs every second and some of these are critical alert streams that need to be processed within milliseconds:

For non-critical data: Set linger.ms around 25 ms.
For critical alerts: Set linger.ms as low as 1-5 ms.
4. Example Configuration

Here’s a sample configuration for both types of agents:

PROPERTIES
# Non-Critical Data Agent
producer.config.non_critical = {
    "batch.size": 16384, # 16 KB
    "linger.ms": 25,
}

# Critical Alert Agent
producer.config.critical_alerts = {
    "batch.size": 1024, # 1 KB for critical data to reduce delay
    "linger.ms": 1,
}
5. Monitoring and Validation

After configuring the producers, it's essential to monitor performance closely:

Throughput: Ensure that the producer is able to handle the expected volume.
Latency: Measure end-to-end latency for both critical alerts and regular data streams.
Error Rate: Check for any unexpected increase in errors or rejections.
6. Adjusting Based on Monitoring

Based on initial monitoring, adjust the settings incrementally:

If non-critical data is causing high throughput but alert latency issues, decrease linger.ms slightly.
If critical alerts are delayed due to excessive batching, increase batch.size for these agents.

#### Closing Summary
and Common Mistakes

By carefully balancing batch.size and linger.ms, we can optimize Kafka producers to handle large volumes of data while maintaining low alert latency. Over-tuning in either direction (too high or too low) can lead to performance degradation, so it’s crucial to have a well-monitored setup.

#### Common mistakes
include setting both parameters too aggressively for one use case at the expense of another or not properly accounting for the criticality of different types of data streams. Always start with conservative settings and gradually tweak based on real-world performance metrics.

**Code:**

```
PROPERTIES
Copy
# Sample Kafka producer configuration snippet
producer.config.non_critical = {
    "batch.size": 16384, # 16 KB
    "linger.ms": 25,
}

producer.config.critical_alerts = {
    "batch.size": 1024, # 1 KB for critical data to reduce delay
    "linger.ms": 1,
}
```

