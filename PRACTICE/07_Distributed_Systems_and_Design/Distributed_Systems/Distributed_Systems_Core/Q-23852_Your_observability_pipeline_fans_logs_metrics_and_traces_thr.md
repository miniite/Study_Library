---
type: "practice"
difficulty: "Deep"
---

# Your observability pipeline fans logs, metrics, and traces through a Kafka topic to downstream detectors, and during a storm one detector falls behind while others keep up. Explain where backpressure should live so a slow detector doesn't drop everyone's telemetry or mask the incident.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Backfills](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/Distributed_Backfills.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

In this scenario, understanding how to manage backpressure in a distributed observability pipeline is crucial. By ensuring that backpressure is isolated to just the slow detector and not affecting the entire system, we can maintain overall telemetry integrity and quickly detect issues.

#### Triaging the Question

The question asks us to explain where backpressure should live so that a single slow detector does not impact the rest of the observability pipeline or obscure an incident. The core issue here is ensuring that the failure of one component (the detector) does not cascade into other parts of the system, and that we can still detect and respond to the storm event.

#### Assumptions
Kafka Topic: We are using Kafka as a distributed log storage and messaging backbone.
Detectors: These are components downstream from Kafka that process logs, metrics, or traces.
Storm Event: An unexpected surge in data volume (logs, metrics) that could overwhelm the detectors.

#### First-Principles Solution

Understanding Backpressure:

Definition: Backpressure is a mechanism to prevent a component from being overwhelmed with more incoming messages than it can process.
Impact: If one detector falls behind and accumulates too much data, it could drop telemetry or mask the incident, thus making it difficult for other components to detect the storm.

Isolating Backpressure:

To avoid this scenario, we need to ensure that backpressure is managed locally within the component itself.
This can be achieved by implementing a local buffer in the detector that limits the incoming data rate and processes it at its own pace.

Buffering Mechanism:

Each detector should have an internal buffer that throttles incoming messages based on its current processing capacity.
When this buffer fills up, the detector can pause or slow down further message consumption until it catches up.

Local vs. Global Backpressure:

Local: Only affects the specific detector and does not impact other detectors or components in the pipeline.
Global: Affects all downstream components, leading to a potential cascade of failures.

#### Implementation
Example:

Using Kafka consumers with configurable buffer settings: max.poll.records controls how many records are fetched at once, and fetch.min.bytes and fetch.max.wait.ms can control the amount of time to wait for more data before fetching.
Implementing a custom backpressure mechanism where each detector monitors its own processing load and adjusts the rate of message consumption accordingly.

#### Closing Summary

In summary, by implementing local buffering in each detector, we ensure that individual components do not impact the entire pipeline. This approach allows us to maintain robust observability during storm events while still detecting issues quickly and accurately.

#### Common Mistakes/Follow-Ups

Mistake: Not isolating backpressure within the detector.

If a detector fails to manage its own load, it could cause a domino effect, leading to other detectors falling behind or dropping critical data.

#### Follow-Up
Question: How does this approach impact performance during normal operations versus storm events?

During normal operations, local buffering may introduce some latency but ensures that all components can handle their share of the load. During storms, the buffer limits ensure that only one component is affected, allowing others to continue functioning.

#### Follow-Up
Question: What are potential trade-offs between this approach and global backpressure mechanisms?

Global backpressure mechanisms might be simpler to implement but could impact the entire system during a storm event, whereas local buffering keeps individual components isolated.

**Comparison:**

```
Detector	Processing Capacity (msg/s)	Buffer Size (messages)
d1	500	10,000
d2	400	8,000
d3	600	12,000
Storm Event Impact
d1: Pauses processing when buffer fills. Other detectors continue to process normally.
d2 and d3: Continue to process without issues due to lower message rate and higher buffer size.
```

**Code:**

```python
# Example of configuring Kafka consumer with buffer settings
consumer = KafkaConsumer(
    bootstrap_servers='localhost:9092',
    auto_offset_reset='earliest',
    max_poll_records=1000,  # Local buffer size
    fetch_min_bytes=1024,   # Minimum bytes to fetch
    fetch_max_wait_ms=500   # Maximum time to wait for more data before fetching
)
```

**Example input / output:**

```text
Input: Kafka topic with high message rate during storm event.

Output:
- Detector d1 processes messages at a pace of 500 msg/s.
- Local buffer in detector d1 is configured to hold up to 10,000 messages.
- If the incoming rate exceeds 500 msg/s, the buffer fills and detection pauses until it catches up.
```

**Data:**

```json
{
  "topic": {
    "name": "storm_logs",
    "partitions": 3,
    "replication_factor": 2,
    "message_rate": "high"
  },
  "detector": {
    "id": "d1",
    "processing_capacity": 500, // Messages per second
    "buffer_size": 10000,       // Local buffer to manage backpressure
    "status": "behind"          // Indicates current processing state
  }
}
```

**Flow:**

1. Yes
2. No
3. Storm Event
4. Kafka Topic
5. Detector d1 Behind?
6. Pause d1 Processing
7. Process Normal

