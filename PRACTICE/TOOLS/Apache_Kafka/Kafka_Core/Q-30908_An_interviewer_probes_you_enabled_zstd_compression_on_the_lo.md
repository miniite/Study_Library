---
type: "practice"
difficulty: "Intermediate"
---

# An interviewer probes: you enabled zstd compression on the log topic and broker CPU spiked. How do you decide between compression ratio and CPU for a high-volume observability pipeline?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Producer Batching](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Producer_Batching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question, we need to understand the trade-off between achieving a higher compression ratio with zstd on Kafka log topics and the increased CPU utilization it might cause, especially in a high-volume observability pipeline.

#### Triaging the Question

The interviewer is probing your understanding of how compression impacts system performance, particularly in terms of CPU usage. They want to see if you can reason through this trade-off from first principles and propose strategies to mitigate potential issues.

#### Assumptions
The log volume is high (millions or billions of messages per day).
The Kafka cluster handles critical observability data.
zstd compression has been enabled on the log topic for space optimization.

#### First-Principles Solution

First, let's break down the core components and their interactions:

Compression: Enabling zstd compression reduces the size of messages before they are written to disk. This can help in reducing storage costs but comes at a cost of CPU usage due to the computational overhead required for encoding and decoding.

CPU Utilization: Increased CPU utilization might affect real-time processing, message latency, and overall cluster performance. High CPU usage can lead to delayed data ingestion or even broker failures if not managed properly.

System Metrics: Monitoring tools like Prometheus, Grafana, or Kafka’s own metrics should be used to track CPU usage, message throughput, and other relevant KPIs.

Trade-off Analysis:

Compression Ratio vs. CPU Utilization: We need to balance the benefits of reduced storage costs against the potential increase in CPU overhead.
Performance Impact on Brokers: High CPU utilization can degrade overall system performance, affecting real-time processing and message delivery times.
Step-by-Step Solution

Initial Investigation:

Check broker metrics for increased CPU usage.
Monitor log throughput and compression rates to understand the impact of zstd.

Capacity Planning:

Assess if the current Kafka cluster has sufficient CPU capacity to handle both normal operations and additional load from compression.

#### Cons
ider upgrading hardware or scaling out the brokers to accommodate higher CPU loads.

Compression Settings Tuning:

Adjust the compression level (zstd levels range from 1 to 22) to find a balance between compression ratio and CPU usage.
Test with different levels to see where the sweet spot lies without causing significant performance degradation.

Parallelism Management:

Use Kafka’s inbuilt parallelism features like replication factor and partitioning strategy to distribute the load evenly across brokers.
Ensure that consumer groups are configured for optimal processing, avoiding overloading any single broker or topic partition.

Real-time Monitoring & Alerts:

Implement continuous monitoring of CPU usage, message latencies, and overall system health.
Set up alerts for critical thresholds to proactively address potential issues before they become problematic.

Impact Analysis:

Conduct load testing with various compression settings to measure their impact on performance.
Analyze the trade-offs between storage savings and real-time processing speed.

Fallback Plan:

Have a plan ready in case further optimization is needed, such as using external tools for post-processing of logs before archiving them.

#### Closing Summary

In conclusion, deciding between compression ratio and CPU utilization involves a careful balance. By monitoring key metrics, tuning compression settings, managing parallelism effectively, and implementing robust monitoring, we can optimize the Kafka cluster to handle high volumes while minimizing performance impact.

#### Common Mistakes/Follow-ups
Overemphasizing Compression: Focusing solely on storage savings without considering CPU overhead.
Insufficient Monitoring: Not setting up proper metrics and alerts to track system health.
Ignoring Broker Load Distribution: Failing to ensure even distribution of load across brokers, leading to bottlenecks.

**Comparison:**

```
Broker	CPU Usage (%)
1	80
2	95
3	75
4	85
Timestamp	Throughput (messages/s)
1672531200	10000
1672534800	15000
```

**Code:**

```python
# Example Python code for monitoring Kafka broker metrics using Prometheus client
from prometheus_client import start_http_server, Gauge

def monitor_kafka_broker_metrics():
    cpu_usage = Gauge('kafka_broker_cpu_usage', 'CPU usage of Kafka brokers')

    # Simulate fetching data from Kafka metrics
    def fetch_cpu_usage(broker_id):
        # Assume this function fetches actual CPU usage metric
        return 80  # Example value

    start_http_server(8000)
    for broker_id in range(1, 5):  # Assuming 4 brokers
        cpu_usage.set(fetch_cpu_usage(broker_id))
```

**Example input / output:**

```text
Input:
- Enable zstd compression on Kafka log topic.

Output:
- Monitor CPU usage and message throughput.
- Tune compression settings to find optimal balance.
- Use parallelism management techniques to distribute load evenly.
- Set up alerts for critical thresholds.
```

**Data:**

```json
{
  "broker_metrics": {
    "cpu_usage": [
      {"timestamp": 1672531200, "value": 80},
      {"timestamp": 1672534800, "value": 90}
    ],
    "message_throughput": [
      {"timestamp": 1672531200, "value": 10000},
      {"timestamp": 1672534800, "value": 15000}
    ]
  }
}
```

**Flow:**

1. Enable zstd compression
2. Tune compression settings
3. Monitor CPU usage andthroughput
4. Adjust parallelism strategies
5. Set up alerts for thresholds
6. Evaluate impact with loadtesting

