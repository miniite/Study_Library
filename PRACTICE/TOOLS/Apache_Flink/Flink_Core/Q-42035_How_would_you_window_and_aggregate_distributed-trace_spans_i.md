---
type: "practice"
difficulty: "Deep"
---

# How would you window and aggregate distributed-trace spans in Flink to compute per-endpoint latency anomalies?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Multi-granularity Windows](<../../../../TOOLS/Apache_Flink/Flink_Core/Multi-granularity_Windows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the task of computing per-endpoint latency anomalies using Apache Flink, we need to break down the problem into manageable steps. We'll start by understanding what is being asked and then delve into how we can achieve this using windowing and aggregation in a distributed-trace scenario.

#### Triage

What the Question Is Asking: The question requires us to compute per-endpoint latency anomalies from distributed traces using Apache Flink's stream processing capabilities. Latency anomalies refer to significant deviations from typical latencies observed for endpoints (e.g., services or microservices).

Key Concepts and Assumptions:

Distributed Traces: Each trace represents a sequence of events captured during the execution of a request.
Endpoints: These are individual components within your system, such as microservices or API endpoints.
Windowing: We need to partition the data into windows for processing.
Aggregation: Aggregating metrics like latency over these windows is crucial.

#### First Principles Solution

Data Ingestion:

Collect and ingest distributed traces from various sources (e.g., Jaeger, Zipkin).

Windowing Strategy:

Use tumbling or sliding windows to process the data in fixed intervals.
Tumbling window: Each window is a distinct interval of time with no overlap.
Sliding window: Each new window overlaps partially with previous ones.

Aggregation and Anomaly Detection:

Calculate latency metrics (mean, standard deviation) for each endpoint over the windows.
Define thresholds to detect anomalies based on historical data or statistical models.

Real-time Processing:

Use Flink's state management and checkpointing mechanisms to ensure fault tolerance and consistency.
Detailed Steps
Step 1: Data Ingestion

We assume we are using a streaming source like Kafka to ingest the traces. Here is an example of how you might set this up in Flink:

JAVA
DataStream<Span> spanStream = env.addSource(new KafkaSource());
Step 2: Windowing Strategy

For simplicity, let's use tumbling windows with a fixed size of 1 minute:

JAVA
DataStream<Windowed<Span>> windowedSpanStream = spanStream.keyBy(span -> span.getEndpoint())
        .window(TumblingEventTimeWindows.of(Time.minutes(1)));
Step 3: Aggregation and Anomaly Detection

We will calculate the mean latency and standard deviation for each endpoint in the window:

JAVA
DataStream<LatencyMetric> metrics = windowedSpanStream
    .aggregate(new LatencyAggregator(), new LatencyResultWindowAssigner())
    .keyBy("endpoint")
    .filter(new AnomalyDetector());

Here, LatencyAggregator and AnomalyDetector are custom classes that handle the aggregation logic and anomaly detection respectively.

Step 4: Real-time Processing

Flink's checkpointing ensures that the state is saved periodically to disk, providing fault tolerance:

JAVA
env.enableCheckpointing(5000);

#### Closing Summary

By using Apache Flink's windowing and aggregation capabilities, we can efficiently compute per-endpoint latency anomalies from distributed traces. The key steps involve ingesting data, defining appropriate windows, performing aggregations, detecting anomalies, and ensuring fault tolerance through checkpointing.

#### Common Mistakes
and Follow-Ups
Incorrect Window Size: Choosing an unsuitable window size can lead to either missing anomalies or excessive noise.
Anomaly Detection Thresholds: Not properly setting thresholds can result in false positives or negatives.
State Management: Mismanagement of state can lead to data loss or inconsistencies. Proper checkpointing and snapshotting are crucial.
Code Blocks
JAVA
// Data Source Setup
DataStream<Span> spanStream = env.addSource(new KafkaSource());

// Window Definition
DataStream<Windowed<Span>> windowedSpanStream = spanStream.keyBy(span -> span.getEndpoint())
    .window(TumblingEventTimeWindows.of(Time.minutes(1)));

// Aggregation and Anomaly Detection
DataStream<LatencyMetric> metrics = windowedSpanStream
    .aggregate(new LatencyAggregator(), new LatencyResultWindowAssigner())
    .keyBy("endpoint")
    .filter(new AnomalyDetector());

// Checkpointing
env.enableCheckpointing(5000);

**Comparison:**

```
2023-10-01
2023-10-01
2023-10-01
2023-10-01
2023-10-01
2023-10-01
2023-10-01
2023-10-01
span1 from
span2 from
span3 from
span4 from
Data Ingestion
Windowing
Aggregation and Anomaly Detection
Window and Aggregation in Apache Flink for Distributed Traces
```

**Code:**

```java
// Data Source Setup
DataStream<Span> spanStream = env.addSource(new KafkaSource());

// Window Definition
DataStream<Windowed<Span>> windowedSpanStream = spanStream.keyBy(span -> span.getEndpoint())
    .window(TumblingEventTimeWindows.of(Time.minutes(1)));

// Aggregation and Anomaly Detection
DataStream<LatencyMetric> metrics = windowedSpanStream
    .aggregate(new LatencyAggregator(), new LatencyResultWindowAssigner())
    .keyBy("endpoint")
    .filter(new AnomalyDetector());

// Checkpointing
env.enableCheckpointing(5000);
```

**Example input / output:**

| Input Span | Output Metric |
|------------|---------------|
| { id: 1, traceId: a, endpoint: serviceA, timestamp: 2023-10-01T12:00:00Z, duration: 500 } | { endpoint: "serviceA", meanLatency: 480, stdDeviation: 20, isAnomaly: false } |

**Data:**

```json
{
  "span": {
    "id": "1234",
    "traceId": "abcd",
    "endpoint": "serviceA",
    "timestamp": "2023-10-01T12:00:00Z",
    "duration": 500
  }
}
```

**Flow:**

1. Ingest Span Data
2. KeyBy Endpoint
3. Tumbling Window (1 min)
4. Avg Latency + Std Dev
5. Filter Anomalies
6. Output Anomaly Metrics

