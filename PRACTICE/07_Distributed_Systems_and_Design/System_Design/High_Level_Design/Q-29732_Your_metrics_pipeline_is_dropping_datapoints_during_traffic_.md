---
type: "practice"
difficulty: "Deep"
---

# Your metrics pipeline is dropping datapoints during traffic spikes and cardinality explosions — walk through how you'd add backpressure aggregation-at-edge and cardinality limits without losing critical alerting metrics.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Backpressure and Load Shedding](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Backpressure_and_Load_Shedding.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of dropping datapoints in a metrics pipeline, we need to implement backpressure mechanisms that aggregate data at the edge before it hits our central processing system and incorporate cardinality limits to prevent overwhelming the system during traffic spikes or cardinality explosions. This approach ensures critical alerting metrics are preserved while maintaining overall system stability.

At a high level, this solution involves three key components:

Edge Aggregation: Processing and aggregating data locally before forwarding it.
Cardinality Limits: Controlling the number of unique identifiers to prevent overwhelming our systems.
Backpressure Mechanisms: Ensuring that downstream consumers handle the data rate appropriately.

Let's dive into each component in detail.

**Comparison:**

```
Component	Purpose
Edge Aggregation	Local processing and aggregation of data to reduce volume before it reaches the central system.
Cardinality Limits	Controlling the number of unique identifiers (e.g., users, products) to prevent overwhelming the system.
Backpressure Mechanisms	Ensuring downstream consumers can handle incoming data rates appropriately to avoid overloading.
Example Input-Output Flow

Input: A stream of 10,000 metric datapoints arriving at a rate that exceeds the processing capacity.

Output:

Buffering Data at Edge: The first 500 points are buffered locally.
Aggregation and Processing: The buffer is processed (aggregated), and the aggregated data is sent to the central system every 5 seconds.
Rate Limiting: After processing, the system waits for new incoming data before sending another batch.

Buffered Data Example:

JSON
Copy
[
    {"metric": "user_activity", "value": 100, "timestamp": 1684972800},
    {"metric": "user_activity", "value": 150, "timestamp": 1684972860}
]
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Edge Aggregation
6. Is Buffer Full?
7. Process and Send to Central
8. Receive New Data
9. Send Aggregated Data
10. Cardinality Limiting
11. Check Unique Identifiers?
12. H
13. Rate Limiting andBackpressure
14. Process Aggregated Data
15. Handle Backpressure

