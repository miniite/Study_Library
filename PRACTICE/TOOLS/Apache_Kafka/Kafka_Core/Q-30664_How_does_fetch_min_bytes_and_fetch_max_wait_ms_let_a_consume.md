---
type: "practice"
difficulty: "Intermediate"
---

# How does fetch.min.bytes and fetch.max.wait.ms let a consumer trade latency for fewer, fatter fetch requests?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Consumer Tuning per Workload](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Consumer_Tuning_per_Workload.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Kafka, fetch.min.bytes and fetch.max.wait.ms are two critical tuning parameters that allow consumers to balance between reducing latency and minimizing the number of fetch requests. These settings provide a mechanism to control how the consumer interacts with the broker and can significantly impact performance.

TL;DR: By adjusting these settings, you can trade off between fetch request frequency (latency) and size (data throughput). Lower fetch.min.bytes values will increase the number of fetch requests but ensure more timely data consumption, while higher fetch.max.wait.ms values allow for larger fetch sizes at the cost of increased latency.

#### Triage

The question is asking about how consumers can use Kafka's configuration parameters to manage their interaction with brokers. Specifically, it focuses on understanding the trade-offs between fetch.min.bytes and fetch.max.wait.ms. The interviewer may be probing your knowledge of consumer behavior in Kafka and how these settings influence data fetching.

#### First-Principles Solution

First, let’s understand what each parameter does:

Fetch Min Bytes (fetch.min.bytes): This sets the minimum number of bytes a fetch request must return before it is considered successful. If the broker cannot fulfill this requirement within the timeout specified by fetch.max.wait.ms, the fetch operation will time out.
Fetch Max Wait Ms (fetch.max.wait.ms): This parameter specifies how long a consumer should wait for additional data from the server to include in the fetched batch before returning.

Here’s a step-by-step reasoning:

Latency vs Throughput:

Lower fetch.min.bytes means the consumer is more likely to send fetch requests frequently, thus reducing latency (time between fetches).
Higher fetch.max.wait.ms allows for larger fetch sizes but increases the time between fetches.

#### Cons
umer Behavior:

When fetch.min.bytes is low and fetch.max.wait.ms is high, the consumer sends more frequent, smaller fetch requests to ensure timely data consumption.
Conversely, when fetch.min.bytes is high and fetch.max.wait.ms is low, fewer but larger fetch requests are sent.

Broker Load:

Lowering fetch.min.bytes increases load on brokers as they handle more frequent small requests.
Raising fetch.max.wait.ms reduces the frequency of request to brokers, potentially decreasing overall broker load.

Scenarios:

In real-time applications where low latency is critical, you might set lower values for both parameters to ensure data is fetched quickly and frequently.
For batch processing jobs or when throughput is more important than latency, you can increase fetch.min.bytes and fetch.max.wait.ms to reduce the number of requests.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By carefully tuning these settings, you can optimize consumer behavior for your specific use case. However, setting these values too low or high without understanding their implications can lead to suboptimal performance:

Under-tuning: Setting fetch.min.bytes and fetch.max.wait.ms too low may result in excessive broker load and higher latency.
Over-tuning: High settings might reduce the number of fetch requests but could also delay data consumption, leading to potential business impacts.

#### Common Mistakes
Ignoring these parameters completely can lead to default behavior that is not optimized for your use case.
Setting values too high or low without understanding their impact on latency and throughput.

#### Follow-up
questions may include:

How do you monitor the performance of consumers with different settings?
What are some best practices for tuning these settings in production?

**Code:**

```java
// Example configuration snippet in Java
Properties props = new Properties();
props.put("fetch.min.bytes", "1");
props.put("fetch.max.wait.ms", 50);
```

**Example input / output:**

```

No specific input-output scenarios are required here, but you could dry-run a simple example:

PLAINTEXT
Copy
Input (consumer configuration): fetch.min.bytes = 1, fetch.max.wait.ms = 50
Output (behavior): Consumer sends frequent small fetch requests with a max wait of 50ms.
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Is 'fetch.min.bytes' low?
7. Send frequent small fetchrequests
8. Is 'fetch.max.wait.ms' high?
9. Send large fetch requests lessfrequently
10. Send moderate fetch requestswith balanced wait time
11. Monitor for optimalperformance

