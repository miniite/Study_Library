---
type: "practice"
difficulty: "Deep"
---

# Your JVM service shows periodic 400ms latency spikes correlated with full GC. How would you distinguish a memory leak from normal churn, and what heap/GC tuning would you try first?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Tail Latency After Expansion](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Tail_Latency_After_Expansion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To diagnose whether a 400ms latency spike in a JVM service is due to a memory leak or normal churn, we need to understand the nature of both and how they manifest in our system. A memory leak occurs when objects are not properly released, leading to an increase in heap size over time, which can cause full GC (Garbage Collection) events to take longer. Normal churn refers to legitimate increases in object allocation and collection due to application logic, but within expected bounds.

We will use diagnostic tools like JVM profiling, logging, and monitoring to distinguish between the two. First, we'll gather detailed metrics on heap usage, garbage collection times, and object lifecycles. Then, we’ll analyze these metrics to identify patterns that indicate either a memory leak or normal churn.

To begin, let's gather some baseline information:

Heap Usage Metrics: Monitor heap size (Eden, Survivor, Old) over time.
GC Logs: Analyze GC logs for full GCs and their duration.
Object Allocation Patterns: Use tools like VisualVM or JProfiler to understand object allocation patterns.

#### Assumptions

The system is running on a production environment with sufficient monitoring in place.
We have access to detailed JVM metrics via metrics collectors such as Prometheus and Grafana.
Step-by-Step Analysis

Heap Size Trends:

Plot the heap size over time. If the heap size increases steadily without bounds, it may indicate a memory leak. However, if there are peaks followed by drops, this could be normal churn.

GC Durations and Frequency:

Look for spikes in full GC durations. If these spikes occur more frequently as the system runs, it could point to memory leaks as the JVM is struggling to reclaim space.

Object Allocation Patterns:

Use profiling tools to identify classes that are being allocated excessively. This can help pinpoint potential memory leak sources.

Thread Dumps and Analysis:

Take thread dumps during the latency spikes and analyze them for long-running threads or blocked states, which could indicate a memory issue.
Initial Heap/GC Tuning

If we determine that normal churn is likely, we might not need to change much. However, if there are signs of a memory leak, initial steps would include:

Adjusting GC Parameters:

Try increasing the MaxNewSize and SurvivorRatio parameters to reduce pressure on Eden space.
Use G1GC (Garbage First) as it is more efficient at managing large heaps.

Monitoring and Alerting:

Set up alerts for heap size, GC duration, and object allocation rates. This will help us quickly respond if issues arise.

Code Review and Profiling:

Conduct a thorough code review to identify potential memory leaks.
Use profiling tools to simulate load conditions that mimic production to catch any new or hidden memory issues.

#### Closing Summary

To distinguish between a memory leak and normal churn, we need detailed metrics and analysis of JVM performance. By monitoring heap size trends, GC durations, object allocation patterns, and thread dumps, we can make informed decisions on whether to tune the garbage collector or address potential code issues. Initial tuning steps would involve adjusting GC parameters and implementing robust monitoring.

#### Common Mistakes

Overlooking baseline data and failing to understand normal system behavior.
Ignoring heap size trends and focusing solely on GC logs.
Not conducting thorough code reviews when memory leaks are suspected.

**Comparison:**

```
Heap Size	GC Durations
2.0 GB	100 ms
2.5 GB	150 ms
3.0 GB	200 ms
4.0 GB	300 ms
```

**Code:**

```java
// Example of setting G1GC parameters in JVM startup options
-Xms2g -Xmx4g -XX:+UseG1GC -XX:MaxNewSize=512m -XX:SurvivorRatio=8
```

**Example input / output:**

```text
Input:
- Heap size over time (X-axis: Time in seconds; Y-axis: Heap Size in GB)
- Full GC duration over time (X-axis: Time in seconds; Y-axis: Duration in ms)

Output:
- If heap size is growing exponentially, it may indicate a memory leak.
- If full GC durations are increasing and correlated with rising heap size, it suggests a potential memory issue.
```

**Data:**

```yaml
# Example of heap size data (in GB)
heap_size:
  time: [0s, 60s, 300s, 900s]
  values: [2.0, 2.5, 3.0, 4.0]

# Example of GC duration data (in ms)
gc_durations:
  full_gc_duration: [100ms, 150ms, 200ms, 300ms]
```

**Flow:**

1. Yes
2. No
3. Increased Duration
4. No Adjustment Needed
5. Code Review
6. Monitor Heap Size
7. Is Heap Increasing Steadily?
8. Identify Memory Leak Source
9. Check GC Durations andFrequency
10. Tune Garbage CollectorParameters
11. Monitor System Behavior
12. Tune Garbage CollectorParameters

