---
type: "practice"
difficulty: "Deep"
---

# A CDC-based streaming load falls behind whenever the source runs a bulk update. How do you keep the consumer from being overwhelmed by that burst?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Batch and Stream Reconciliation](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Batch_and_Stream_Reconciliation/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a CDC (Change Data Capture)-based streaming load falling behind during a bulk update, we need to ensure that the system can handle the surge in data without overwhelming its consumers. The core idea is to balance the processing load by optimizing the consumer's handling of large bursts of data.

This question probes your understanding of batch vs. stream processing and your ability to design robust solutions for managing data streams under varying conditions. We'll assume a typical scenario where a streaming system processes changes from a database in real-time but gets overwhelmed when a bulk update occurs, causing delays.

First, let's break down the key steps:

Understand the source of the bulk updates.
Design a mechanism to handle bursts without overloading consumers.
Ensure fault tolerance and consistency in data processing.

To address this, we can implement a buffer-based approach that smooths out the spike in incoming data. This involves temporarily storing the data locally or using a message queue, which allows for controlled consumption by downstream processes.

Let's reason step-by-step:

Identify the Burst: Detect when a bulk update is occurring. This could be based on the volume of changes, frequency, or specific patterns.
Buffer Mechanism: Introduce a buffer that temporarily holds incoming data until it can be processed without overwhelming consumers.
Dynamic Sizing: Adjust the size and capacity of this buffer dynamically to handle bursts more effectively.

#### Cons
umer Rate Control: Implement rate limiting on consumers to prevent them from processing too much data at once.

Now, let's go through these steps in detail:

Detecting Bulk Updates:

Monitor the change feed for high volumes of changes over a short period.
Use metrics like event counts per second or total events within a fixed time window.

Buffer Mechanism:

Introduce an intermediate storage layer, such as a message queue (e.g., Kafka, Redis Streams) or local buffers in the streaming system itself.
When a bulk update is detected, push all incoming data into this buffer instead of directly processing it.

Dynamic Sizing:

Configure the buffer size to handle bursts. This can be done based on historical data or dynamically adjusted using machine learning techniques to predict and adapt to varying loads.
Ensure that the buffer has enough capacity to absorb the surge without dropping messages.

#### Cons
umer Rate Control:

Implement rate limiting for consumers, which means they consume data at a controlled pace rather than in bulk.
Use backpressure mechanisms where consumers signal upstream producers to slow down when the buffer is full or processing lag increases.

Here’s how you might implement these steps using a simplified example with a streaming architecture:

Source CDC Stream
Buffer Queue

#### Cons
umer 1

#### Cons
umer 2

#### Cons
umer 3
Detection Mechanism

In this diagram, the F node represents a mechanism to detect bulk updates and trigger buffer behavior.

