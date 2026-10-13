---
type: "practice"
difficulty: "Deep"
---

# Explain the difference between at-least-once and exactly-once delivery for an alert stream, and why for correlation you might actually prefer at-least-once with downstream dedup. When does that reasoning fail?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Processing Time vs Event Time](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Processing_Time_vs_Event_Time.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a real-time streaming architecture for AIOps, ensuring timely and accurate alerts is critical. The core issue we're dealing with is guaranteeing either exactly-once or at-least-once delivery of alert messages in such systems. Exactly-once processing ensures that each message is processed exactly once, which prevents duplicates and missing events. At-least-once delivery means that every message will be processed at least once but could potentially lead to duplicate processing.

For our specific scenario involving correlation, let's delve into why you might prefer at-least-once with downstream deduplication over exact-once. The key differentiator lies in the trade-offs between consistency and availability under high load or failure conditions.

**Comparison:**

```
Delivery Type	Guarantees	Trade-offs
Exactly-once	No duplicates, no missing events	Higher complexity; potential for retries leading to delays
At-least-once	Simpler implementation, fewer infrastructure requirements	Duplicates possible, need downstream deduplication
```

**Code:**

```
MERMAID
Copy
sequenceDiagram
    participant Producer
    participant StreamProcessor
    participant Correlator
    participant DownstreamDedup
    participant Consumer
    Producer ->> StreamProcessor: Produce alert
    StreamProcessor -->> Correlator: Process alert
    Correlator -->> DownstreamDedup: Correlated alert (at-least-once)
    DownstreamDedup -->> Consumer: Deduplicate and deliver to consumer
```

**Example input / output:**

```text
An input alert from a server experiencing high CPU usage might look like the JSON snippet above. When processed by the correlator, it could generate correlated alerts that are then sent to a downstream deduplication layer.

For instance:
- **Input:** High CPU usage on `server1.example.com`
- **Correlated Output (at-least-once):** Correlated events such as "Server under heavy load" or "Potential DDoS attack"
- **Deduplicated and Delivered Output:** Ensure only unique alerts reach the consumer.
```

**Data:**

```json
{
  "alert": {
    "id": "ALERT-123",
    "timestamp": "2023-10-01T14:00:00Z",
    "source": "server1.example.com",
    "event_type": "high_cpu_usage"
  }
}
```

**Flow:**

1. Alert Production
2. Stream Processing
3. Correlation (at-least-once)
4. Deduplication
5. Consumer Delivery

