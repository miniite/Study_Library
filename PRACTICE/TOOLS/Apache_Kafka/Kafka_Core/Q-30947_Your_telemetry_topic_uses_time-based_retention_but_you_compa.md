---
type: "practice"
difficulty: "Deep"
---

# Your telemetry topic uses time-based retention but you compact a separate service-state topic. How do you decide which pattern fits which stage of the AIOps pipeline?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Log Compaction](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Log_Compaction.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In an AIOps environment, choosing between time-based retention and compaction for different stages in the pipeline is crucial to optimize storage efficiency while maintaining necessary data for analysis. For a telemetry topic that uses time-based retention and a service-state topic with compaction, we need to understand how these patterns fit into the broader context of the pipeline.

#### Triaging the Question

The question asks about deciding which pattern (time-based retention or compaction) should be used in different stages of an AIOps pipeline. Specifically:

Time-based Retention: Keeps data for a fixed period, typically useful for long-term storage and analysis.
Compaction: Repeatedly removes older records to reduce the size of the topic, ideal for high-volume production topics.

#### Assumptions
The telemetry topic is collecting large volumes of real-time data over time.
The service-state topic contains critical state changes that need to be preserved indefinitely or until explicitly deleted by a business rule.

#### Solution
from First Principles
Telemetry Topic: Time-Based Retention

Telemetry data, such as metrics collected from various services and infrastructure components, often needs to be stored for historical analysis. Time-based retention ensures that this data is kept for a specific period, allowing us to analyze trends over time without overwhelming storage space.

Storage Requirements: Telemetry data can grow exponentially, so it's important to retain only the recent data needed for analysis.
Query Performance: Historical queries on old data are less frequent than real-time monitoring, making time-based retention more efficient.
Cost Management: Long-term storage is expensive and managing older data through compaction could be wasteful.
Service-State Topic: Compaction

Service-state data, such as current service health, configurations, and operational statuses, needs to be retained indefinitely because it reflects the current state of critical systems. Compaction helps manage this high volume efficiently:

Continuous Updates: Service states change frequently but still require long-term retention.
Space Optimization: Compacting old records reduces storage overhead without losing historical context through periodic compaction.

#### Closing Summary
and Common Mistakes

In summary, the telemetry topic should use time-based retention to balance storage efficiency with the need for recent data analysis. The service-state topic should employ compaction to manage high volume changes efficiently while retaining crucial state information indefinitely.

#### Common mistakes
include:

Over-relying on compaction in all scenarios, which can lead to excessive write amplification.
Ignoring the necessity of long-term retention for critical operational metrics, risking loss of essential historical data.

**Comparison:**

```
Topic	Retention Policy	Purpose
Telemetry	Time-based	Historical data analysis and monitoring
Service-state	Compaction	Continuous service health and state tracking
```

**Code:**

```python
# Example code snippet for configuring Kafka topics with different retention policies

from kafka.admin import NewTopic, AdminClient

def configure_topic(topic_name, config):
    admin_client = AdminClient({'bootstrap.servers': 'localhost:9092'})

    new_topics = [NewTopic(name=topic_name, num_partitions=1, replication_factor=3, **config)]

    admin_client.create_topics(new_topics)

# Time-based retention
configure_topic("telemetry", {"retention.ms": 86400000}) # Retain for 24 hours

# Compaction
configure_topic("service-state", {"delete.retention.ms": 1296000000, "min compaction lags": 3})
```

**Example input / output:**

```text
Input:
- Telemetry topic configuration request with time-based retention.
- Service-state topic configuration request for compaction.

Output:
- Telemetry topic configured with a retention period of 24 hours and delete policy applied.
- Service-state topic configured to compact old records after 180 days, retaining at least 3 lags for compaction.
```

**Data:**

```json
{
    "telemetry": {
        "retention.ms": 86400000,
        "cleanup.policy": ["delete"]
    },
    "service-state": {
        "compression.type": "gzip",
        "delete.retention.ms": 1296000000,
        "min.compaction.lags": 3
    }
}
```

**Flow:**

1. Telemetry Topic
2. Time-Based Retention
3. Historical Analysis &Monitoring
4. Service-State Topic
5. Compaction
6. Continuous State Tracking

