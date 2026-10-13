---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd guarantee that the exact feature values used for a prediction are logged alongside that prediction across a distributed serving fleet, and why reconstructing them after the fact from the offline store is unreliable. What's the throughput and storage design for this at high QPS?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Distributed Tracing](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/Distributed_Tracing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that exact feature values used in predictions are logged alongside those predictions across a distributed serving fleet, we need to implement a system where each prediction request and its corresponding feature values are stored atomically and persistently. This is crucial to avoid the unreliability of reconstructing these feature values from an offline store after the fact. Here’s how we can achieve this:

Atomic Logging: Each prediction request should include all necessary feature values in a log entry that is written atomically.
Logging Mechanism: We need a high-throughput, low-latency logging mechanism to handle the large volume of requests at high QPS (queries per second).

#### Cons
istent Storage Design: The storage design must be robust and scalable to ensure reliable data retrieval and minimal latency.

Let’s dive into the details:

#### Triaging the Question
Core Requirement: Ensure feature values are logged alongside predictions.
Probing Points: Understanding of distributed systems, logging mechanisms, and storage design. The interviewer is probing your knowledge of handling high QPS in a scalable manner while ensuring consistency and reliability.

#### Assumptions
The serving fleet consists of multiple microservices or containers that perform predictions.
The feature values are dynamically generated based on the input data and may change over time.
We need to log these feature values for auditing, debugging, or historical analysis purposes.

#### First Principles Solution
1. Request Logging

Each prediction request should be accompanied by all necessary feature values in a log entry that is written atomically. This ensures that the log entry and the corresponding prediction are always consistent.

Request
Feature Values
Log Entry
Persisted Log
2. Throughput and Storage Design
High Throughput Logging Mechanism: Use a distributed logging system like Apache Kafka or Cloud Pub/Sub to handle high QPS.
Storage Scalability: Implement a scalable storage solution such as Amazon S3, Google Cloud Storage, or a distributed database like Cassandra for storing the logs.
3. Consistent Logging
Timestamps and Metadata: Include timestamps and metadata in each log entry to ensure consistency across multiple requests.
Replication: Use replication strategies to ensure high availability and fault tolerance of the logging system.
Throughput Considerations
At a QPS of, say, 10,000 predictions per second, we need a logging mechanism that can handle this throughput without significant latency or loss of data.
Kafka is well-suited for this due to its built-in scalability and fault tolerance. Each log entry can be processed in parallel by multiple consumers.
Storage Design
Partitioning: Partition logs based on time or request ID to ensure even distribution across storage nodes.
Indexing: Maintain an index of logged predictions to enable efficient querying and retrieval.
RetentionPolicy: Define a retention policy for logs to manage storage costs while maintaining necessary historical data.

#### Example
Storage Schema
YAML
log_entry:
  - id: string (unique identifier for the log entry)
  - timestamp: datetime
  - request_id: string (identifier for the prediction request)
  - feature_values: array (array of feature values used in the prediction)
  - prediction_result: object (result of the prediction)

index:
  - request_id: string (identifier for the prediction request)
  - timestamp: datetime
Reliability and Failure Modes
Latency: Ensure that the logging mechanism has minimal latency to avoid affecting prediction performance.
Redundancy: Use multiple replicas to ensure data availability in case of node failures.
Monitoring and Alerts: Implement monitoring and alerting for log entry processing times and storage usage.

#### Closing Summary

In summary, ensuring feature values are logged alongside predictions requires a robust logging mechanism with high throughput and consistent design. By using distributed systems like Kafka for logging and scalable storage solutions like S3 or Cassandra, we can handle high QPS while maintaining data integrity and reliability.

#### Common Mistakes/Follow-Ups
Mistake: Failing to consider the latency impact on prediction performance.

#### Follow-Up
Discussing how to balance logging overhead with system performance. How do we ensure that our logging mechanism does not become a bottleneck?

**Comparison:**

| Feature Value | Description |
|---------------|-------------|
| id            | Unique identifier for the log entry |
| timestamp     | Timestamp when the prediction was made |
| request_id    | Identifier for the prediction request |
| feature_values| Array of values used in the prediction |
| prediction_result | Result of the prediction including label and confidence score |

**Code:**

```python
# Example Python code for logging in Kafka
from kafka import KafkaProducer

def log_prediction(request_id, feature_values, prediction_result):
    producer = KafkaProducer(bootstrap_servers='kafka-bootstrap-server')

    # Prepare the log entry
    log_entry = {
        "request_id": request_id,
        "feature_values": feature_values,
        "prediction_result": prediction_result
    }

    # Send the log entry to Kafka
    future = producer.send('predictions', value=log_entry)
    result = future.get(timeout=10)  # Wait for confirmation

    print(f"Logged prediction for request {request_id}")
```

**Example input / output:**

```text
Input:
Request ID: req-123456789
Feature Values: [1.2, 3.4, 5.6]

Output:
Logged prediction for request req-123456789 with feature values [1.2, 3.4, 5.6]
```

**Data:**

```yaml
log_entry:
  id: abc-def-ghi
  timestamp: 2023-10-01T14:00:00Z
  request_id: req-123456789
  feature_values: [1.2, 3.4, 5.6]
  prediction_result: {label: "positive", confidence: 0.9}
```

**Flow:**

1. Request
2. Feature Values
3. Log Entry
4. Kafka Topic
5. Persisted Log
6. Storage

