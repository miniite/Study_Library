---
type: "practice"
difficulty: "Expert"
---

# An interviewer asks you to reason about tail latency at fleet scale: p99.9 produce latency correlates with GC pauses segment rolls and page-cache pressure. How do you architect the broker fleet (heap GC choice IO scheduling instance types) to bound tail latency as a platform guarantee?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Kafka Capacity and Cost](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Kafka_Streams_and_Event_Time/Kafka_Capacity_and_Cost.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To reason about tail latency in Kafka at scale, particularly focusing on p99.9 produce latency and its correlation with garbage collection (GC) pauses, segment rolls, and page-cache pressure, we need to architect the broker fleet carefully by choosing appropriate heap GC algorithms, IO scheduling policies, and instance types. This involves a deep understanding of how these components interact to impact performance and ensuring that tail latencies are bounded as a platform guarantee.

**Comparison:**

| Broker Configuration | Parameters                    | Values                |
|-----------------------|-------------------------------|-----------------------|
| Heap Size             | Initial/Max                   | 4GB / 8GB             |
| GC Strategy           | Algorithm                     | G1GC                  |
| GC Latency Target     | MS                            | 50ms                  |
| Instance Type         | For High Traffic              | r5.xlarge             |

**Code:**

```
MERMAID
Copy
graph TD;
    A[Broker Fleet] --> B[Heap GC Algorithm]
    A --> C[IO Scheduling Policy]
    A --> D[Instance Types]
    B --> E[GCLatency]
    C --> F[Disk Latency]
    D --> G[System Resources]
    E --> H[P99.9 Produce Latency]
    F --> H
    G --> H
```

**Example input / output:**

```text
Input:
- High-producer traffic pattern with frequent large batch messages
- Desired p99.9 latency < 200ms

Output:
- Configured brokers using G1GC with a target GC pause of 50ms, ensuring minimal impact on tail latency.
- IO scheduling set to CFQ (Completely Fair Queuing) for balanced I/O load distribution.
- EC2 instances selected based on the workload profile: c5.large for moderate traffic and r5.xlarge for high-traffic scenarios.

Request:
Broker configuration request for a new Kafka cluster with 10 brokers, each having 8GB heap size managed by G1GC.
Response:
Broker configurations successfully set up with G1GC algorithm, CFQ scheduler, and appropriate EC2 instance types to meet the latency requirement.
```

**Data:**

```yaml
# Example Kafka broker configuration snippet for heap GC settings
heap_GC: {
  initial_heap_size: 4g,
  max_heap_size: 8g,
  gc_strategy: "G1GC",
  gclatency_target_ms: 50
}
```

**Flow:**

1. High-producer traffic
2. Segment Roll
3. GCLatencyIncrease
4. P99.9 LatencyImpact
5. TailLatencyBound
6. SystemLoadIncrease
7. PageCachePressure
8. P99.9 LatencyImpact

