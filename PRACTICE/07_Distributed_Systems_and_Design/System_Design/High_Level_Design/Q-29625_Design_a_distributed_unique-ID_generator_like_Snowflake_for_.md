---
type: "practice"
difficulty: "Deep"
---

# Design a distributed unique-ID generator (like Snowflake) for a system creating 1M IDs/sec across data centers — how do you keep IDs roughly time-ordered and collision-free?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Audit and Compliance Platforms](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Audit_and_Compliance_Platforms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a high-level distributed unique-ID generator that can handle 1 million IDs per second (1M/sec) across multiple data centers, we need to ensure both time-ordering and collision-freeness of generated IDs. This is akin to the problem tackled by Snowflake's ID generation algorithm but with additional constraints due to distributed nature.

The key challenge here involves maintaining global uniqueness while ensuring efficient and scalable ID assignment. The core idea revolves around leveraging a combination of sequential numbers, timestamps, and random components to achieve this goal. Let’s break down the design step-by-step:

Understand the Question: The interviewer is probing for an understanding of how to design a distributed system that can generate unique IDs in real-time with minimal latency and high throughput. The focus will be on handling large-scale operations across multiple data centers.

Key Assumptions:

We have control over the hardware in each data center.
Network latencies between data centers are variable but within a known range.
Each data center has its own clock, which may not be perfectly synchronized with others due to network delays and computational latencies.

#### First-Principles Solution

Time Component: Use millisecond timestamps as the base for ID generation to ensure time-ordering locally.
Sequence Numbers: Combine a sequence number that increments monotonically within each data center. This sequence number should be unique even if two IDs are generated in quick succession due to network delays or local processing delays.
Random Component: Introduce randomness (e.g., a 12-bit random value) to break ties and ensure uniqueness across different data centers.

Detailed Design:

Timestamps: Use the Unix timestamp truncated to millisecond precision as part of the ID. This ensures that IDs are roughly time-ordered within each second.
Local Sequence Number: Each data center maintains a monotonically increasing sequence number. This can be implemented using atomic counters or incrementing integers stored in a database accessible only within the data center.
Random Value: Add a 12-bit random value to ensure that IDs are unique even if two timestamps and sequences overlap. This randomness is crucial for breaking ties caused by network delays.

Synchronization Mechanisms:

To maintain consistency, periodically sync the local clocks across data centers using mechanisms like NTP (Network Time Protocol). This ensures that timestamps can be compared accurately.
Implement a heartbeat mechanism to detect and correct clock drifts in real-time. If a data center's clock is significantly off, it can request a timestamp update from another healthy node.

Handling Failures:

Use distributed leader election algorithms (e.g., Raft) to elect a primary node that coordinates the sequence numbers across all nodes.
Implement retry mechanisms and fallbacks in case of network partitions or node failures.

Optimizations and Trade-offs:

#### Cons
ider using hardware-based timestamping units like Intel’s Time Stamp Counter (TSC) to get more precise timestamps.
Optimize database access patterns for the sequence numbers to minimize latency.
Ensure that the random component is truly unpredictable and unique across all data centers.

**Comparison:**

```
Data Center ID	Sequence Number	Timestamp
DC1	12345	1640995200000
DC2	67890	1640995200001
```

**Flow:**

1. No
2. Yes
3. Generate Timestamp
4. Is Clock Drift?
5. Increment Sequence Number
6. Add Random Value
7. Request Synchronized Timefrom Peer
8. Sync Clocks Periodically
9. Retry or Fall Back on Sync

