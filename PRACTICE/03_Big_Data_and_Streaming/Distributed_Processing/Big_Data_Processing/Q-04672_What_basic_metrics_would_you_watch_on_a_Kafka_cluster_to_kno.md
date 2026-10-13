---
type: "practice"
difficulty: "Intermediate"
---

# What basic metrics would you watch on a Kafka cluster to know it is healthy?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Consumer Lag Diagnosis](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Consumer_Lag_Diagnosis.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure a Kafka cluster is healthy, we need to monitor several key metrics that provide visibility into both the health of individual nodes and the overall performance and reliability of the cluster. At an intermediate level, these metrics should cover latency, throughput, error rates, and resource utilization.

#### Triage

The question asks for basic metrics to ensure a Kafka cluster is healthy. The interviewer is probing your understanding of Kafka's architecture and key performance indicators (KPIs) that reflect the health and efficiency of the system. Key assumptions include:

You are familiar with Kafka’s operational characteristics.
The focus is on real-time monitoring tools such as Prometheus, Grafana, or other observability solutions.

#### First Principles Solution

Latency Metrics:

Produce Latency: Measures how long it takes for messages to be produced and sent to the cluster.
Fetch Latency: Monitors how long consumers take to fetch data from Kafka brokers.

Throughput Metrics:

Messages Produced Per Second (P/s): Tracks the number of messages being added to the cluster per second.
Messages Consumed Per Second (C/s): Measures the rate at which consumers are pulling messages from the brokers.

Error Rates:

Producer Errors: Monitors errors occurring during message production, such as retries or failures.

#### Cons
umer Offsets Lagging: Indicates how much behind real-time data consumption is lagging.
Network Errors: Tracks any network-level issues that could affect the flow of messages.

Resource Utilization:

CPU and Memory Usage: Ensures brokers have enough resources to handle current load without running into performance bottlenecks.
Disk I/O: Measures read/write operations on disk, indicating potential storage-related issues.

Partition Balance Metrics:

Leader Elections: Monitors leader election events and their frequency.
Replication Latency: Tracks how long it takes for data to replicate across replicas in the same partition.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, a healthy Kafka cluster should exhibit low latency, high throughput, minimal error rates, and efficient resource utilization. Monitoring these metrics with tools like Prometheus and Grafana can help you proactively address issues before they impact critical services.

#### Common mistakes
include:

Focusing solely on throughput without considering latency.
Overlooking network errors that could be indicative of broader connectivity issues.
Ignoring disk I/O patterns, which might indicate storage capacity constraints or performance bottlenecks.

#### Follow-ups
might include discussing how to set up alerts based on these metrics and the trade-offs between different monitoring tools and strategies for maintaining a balanced Kafka cluster.

**Comparison:**

```
Metric	Description
Produce Latency	Time taken to produce and send messages
Messages P/s	Number of messages produced per second
Consumer Errors	Rate of errors during message consumption
CPU Usage	Percentage of CPU utilized by brokers
```

**Code:**

```python
# Example of Prometheus query for Consumer Offsets Lagging (PROMQL)
consumer_lag_query = 'sum(max by (topic, partition) (rate(kafka_consumer_offset_lag_sum[1m])))'
```

**Example input / output:**

```
Dry Run of Latency Metrics
PLAINTEXT
Copy
Produce Latency (ms): 100.2
Fetch Latency (ms): 200.5
```

**Data:**

```
Consumer Offset Lag: JSON snippet from a metrics endpoint
JSON
Copy
{
  "partitionOffsets": [
    {"topic": "topicA", "partition": 0, "offset": 123456789},
    {"topic": "topicB", "partition": 1, "offset": 987654321}
  ]
}
```

**Flow:**

1. Produce Latency
2. Below Threshold OK
3. Messages P/s
4. Above Threshold Warning
5. Consumer Errors
6. Above Threshold Critical
7. CPU Usage
8. Below Threshold OK

