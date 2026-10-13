---
type: "practice"
difficulty: "Intermediate"
---

# How would you configure a Kafka producer for maximum throughput? Discuss batching and linger.ms.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Producer Batching](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Producer_Batching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To configure a Kafka producer for maximum throughput, we need to focus on optimizing several key parameters, particularly those related to batching and linger.ms. Batching allows multiple messages to be combined into a single batch request, reducing the number of network round trips and improving overall performance. linger.ms controls how long a message is held in memory before it's sent out as part of a batch, further optimizing throughput.

The core idea here is to strike a balance between message batching and latency. A higher value for linger.ms allows more messages to accumulate into a single batch, but this increases the potential delay in sending individual messages. Conversely, setting linger.ms too low can lead to frequent small batches, which may not fully utilize network capacity.

Let's break down the process step-by-step:

#### Triage
Understanding the Question

The question is asking us to configure a Kafka producer for maximum throughput by focusing on batching and linger.ms. The interviewer wants to ensure you understand how these parameters interact with each other and affect performance. We will assume that we are dealing with a producer that sends messages in real-time, like monitoring data or streaming logs.

#### Assumptions
Network latency is minimal.
There is no significant loss of data due to retransmissions.
The system can handle occasional message delays if necessary.

#### First Principles Solution
Batching: Batching multiple messages into a single batch request reduces the overhead of individual network requests. This means setting an appropriate batch size and using linger.ms effectively.
linger.ms: Controls how long the producer should wait for more messages before sending out a batch. A higher value can lead to larger batches, but may increase latency.
Step-by-Step Configuration

Batch Size:

Producers often use the default batch size of 16 messages.
Increasing this number can reduce network overhead and improve throughput, but setting it too high might not be optimal due to increased memory usage and potential delays for smaller messages.

linger.ms:

This parameter controls how long the producer waits before sending out a message as part of a batch. A value of 0 means immediate delivery.
Setting linger.ms to a non-zero value (e.g., 1-5 ms) allows more messages to accumulate into a single batch, reducing network requests and improving throughput.

Compression:

Enabling compression can further reduce the size of each message, allowing more data to be sent in a single batch.
Common compressions are gzip, snappy, or lz4.

#### Example
Configuration
PROPERTIES
# Kafka Producer Properties
bootstrap.servers=your_kafka_brokers
batch.size=32768  # Default is 16, doubled to allow more messages in a batch
linger.ms=5       # Allow up to 5 ms for additional messages before sending
buffer.memory=33554432  # Buffer size to store messages in memory (default is 33MB)
compression.type=producer# Compress each record sent from the producer
acks=all          # Ensure all brokers acknowledge receipt of the message

#### Closing Summary
and Common Mistakes/Follow-Ups

By carefully configuring the batch size, linger.ms, and compression settings, you can optimize Kafka producers for maximum throughput. Common mistakes include setting linger.ms too high, leading to increased latency, or not compressing messages adequately, which reduces the number of bytes sent per request.

During an interview, you might be asked follow-up questions such as:

How do you measure and monitor the performance of your producer?
What are the trade-offs between throughput and latency in this context?

