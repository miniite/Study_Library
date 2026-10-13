---
type: "practice"
difficulty: "Expert"
---

# Your p99 is fine but p99.9 is terrible and only for one endpoint. How do you methodically hunt the cause — GC pauses, lock contention, a slow dependency, or a single hot partition?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Tail Latency After Expansion](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Tail_Latency_After_Expansion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To diagnose why your system's p99.9 latency is terrible specifically for one endpoint while p99 is fine, we need to methodically hunt down potential bottlenecks such as garbage collection (GC) pauses, lock contention, slow dependencies, or a single hot partition. The key idea is to systematically isolate and verify each potential issue using performance monitoring tools and data.

First, let's triage the problem. We are looking for intermittent spikes in latency that significantly degrade p99.9 but not p99. This suggests an issue with tail latency, likely caused by a specific request pattern or dependency. Common causes include GC pauses, which can occur during large object allocations and collections; lock contention, leading to thread starvation; slow dependencies like external services; and hot partitions, where some data is accessed far more frequently than others.

To methodically hunt the cause:

Monitor and Log: Use APM tools (e.g., New Relic, Datadog) to monitor latencies in real-time for that specific endpoint. Capture logs with timing information, including start and end times of requests. This helps identify any patterns or correlations.

Gather Metrics: Collect detailed metrics on GC activity, thread usage, and dependency response times. For example, JVM tools like VisualVM can provide insights into garbage collection pauses.

Analyze Logs: Look for any anomalies in the logs around the time of high latency events. This could indicate issues with specific parts of the code or external dependencies.

Profiling Tools: Use profiling tools to analyze CPU and memory usage during periods of high latency. Tools like VisualVM, JProfiler, or even custom-built profilers can help identify hotspots in your application.

Dependency Isolation: Temporarily disable or isolate the problematic dependency and monitor if the p99.9 latency improves. This helps confirm whether an external service is a bottleneck.

Partition Analysis: If the endpoint processes data, analyze which parts of the data are accessed more frequently to identify hot partitions. Use techniques like caching or rate limiting on high-accessed data to mitigate this issue.

GC Optimization: Review garbage collection settings and logs for any GC pauses that correlate with latency spikes. Adjust JVM parameters if necessary, such as increasing heap size or changing collection algorithms.

Lock Contention Monitoring: Use tools like Java Flight Recorder (JFR) to monitor thread contention. High contention can be a sign of lock issues or race conditions.

A/B Testing: Implement A/B testing by introducing changes incrementally and comparing the impact on latency. This helps isolate code changes from other variables.

Root Cause Analysis: After identifying potential causes, perform root cause analysis to understand why these issues are occurring. For example, if GC pauses are an issue, investigate the reasons behind large object allocations or frequent collections.

#### Closing summary
By systematically monitoring and analyzing various aspects of your system, you can identify the specific cause of high p99.9 latency for that endpoint. Common mistakes include jumping to conclusions too quickly without thorough data collection and analysis. Follow a methodical approach to ensure accurate diagnosis and effective mitigation strategies.

**Comparison:**

```
Metric	Value
Average Latency	10ms
p99 Latency	20ms
p999 Latency	3000ms
GC Pauses (Frequency)	2 per hour
Lock Contention Rate	5%
```

**Code:**

```

// Example code snippet to monitor GC activities using VisualVM public class GarbageCollectorMonitor { public static void main(String[] args) throws Exception { System.gc(); // Suggesting garbage collection Thread.sleep(1000); // Sleep for 1 second

CODE
Copy
    Runtime runtime = Runtime.getRuntime();
    long beforeFreeMemory = runtime.freeMemory();

    // Perform some operations here that might trigger GC

    long afterFreeMemory = runtime.freeMemory();
    if (beforeFreeMemory - afterFreeMemory > 1024 * 1024) {
        System.out.println("Garbage collection detected!");
    }
}

}

CODE
Copy

```shell
# Example command to start Java Flight Recorder for thread contention analysis
jcmd <pid> JFR.start name="Thread Contention" settings="{duration=PT5M, filename=/tmp/threads.jfr}"
```

**Example input / output:**

```text
Input:
- Endpoint A with high p99.9 latency
- No issues with p99

Output:
- Real-time monitoring of endpoint A shows frequent GC pauses and thread contention.
- Profiling tools identify large object allocations during certain request patterns.
- Disabling external dependency temporarily reduces p99.9 latency significantly.
```

**Data:**

```json
{
  "endpointMetrics": {
    "averageLatency": 10ms,
    "p99Latency": 20ms,
    "p999Latency": 3000ms
  },
  "gcPauses": [
    {"time": 16:00, "duration": 50ms},
    {"time": 16:30, "duration": 70ms}
  ],
  "threadUsage": {
    "cpuUtilization": 80%,
    "lockContentionCount": 20
  }
}
```

**Flow:**

1. Identify High Latency Endpoint
2. Monitor and Log
3. Gather Metrics
4. Analyze Logs
5. Profiling Tools
6. Dependency Isolation
7. Partition Analysis
8. GC Optimization
9. Lock Contention Monitoring
10. A/B Testing
11. Root Cause Analysis

