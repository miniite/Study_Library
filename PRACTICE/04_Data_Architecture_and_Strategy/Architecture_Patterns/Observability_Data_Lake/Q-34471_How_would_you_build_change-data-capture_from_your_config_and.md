---
type: "practice"
difficulty: "Deep"
---

# How would you build change-data-capture from your config and CMDB into the lake so incident timelines include infrastructure changes?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Mesh vs Central Observability Lake](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Mesh_vs_Central_Observability_Lake.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build change-data-capture (CDC) from configuration management and IT asset management systems into an Observability Data Lake, we need to ensure that infrastructure changes are recorded in near-real-time. This is crucial for incident timelines as it helps in understanding the context of issues by identifying recent changes that might have impacted the system. The core idea here is to capture change events (e.g., configuration updates, infrastructure modifications) and stream them into a central repository where they can be correlated with other observability data.

#### Triage

The question asks how to implement CDC from config management and CMDB systems into an Observability Data Lake. This involves several key steps:

Data Sources: Identify the sources of configuration changes (e.g., configuration management tools, IT asset databases).
Change Events: Capture specific events related to infrastructure changes.
Real-Time Streaming: Ensure that these events are captured in real-time or near-real-time.
Processing and Transformation: Transform raw data into a format suitable for storage in the Data Lake.
Data Ingestion: Stream processed change events into the Observability Data Lake.

#### First Principles Solution
Identify Change Events:
Configuration management tools often provide APIs or event streams that can be tapped to capture changes (e.g., Ansible, Chef, Puppet).
CMDB systems typically have audit logs and change histories that need to be ingested.
Capture Change Events:
Use Change Data Capture (CDC) techniques to monitor these sources in real-time or near-real-time.
For example, tools like Debezium can capture changes from databases, while Kafka can act as a message broker for real-time data streams.
Transformation and Enrichment:
Transform raw change events into a standard format that is suitable for storage in the Data Lake.
Add contextual information such as timestamps, user details, and affected resources.
Ingestion into the Data Lake:
Use a streaming platform like Apache Kafka or AWS Kinesis to ingest these transformed events.
Store them in a scalable, low-latency data store within the Observability Data Lake.

#### Implementation
Steps
Set Up Change Event Capture:
For configuration management tools, use their built-in CDC mechanisms if available.
For CMDB systems, set up log collection and parsing to capture change events.
Real-Time Streaming:
Use Kafka/Kinesis to stream these change events in real-time or near-real-time.
Transformation Pipeline:
Develop a transformation pipeline using tools like Apache Beam or AWS Glue.
Ensure that the data is enriched with necessary metadata (e.g., event type, timestamp).
Data Ingestion and Storage:
Store transformed events in the Data Lake using technologies such as Hadoop Distributed File System (HDFS) or Amazon S3.

#### Cons
ider using a time-series database like TimescaleDB for additional analytics.

#### Closing Summary

By implementing CDC from configuration management and CMDB systems, we can ensure that infrastructure changes are captured and stored in an Observability Data Lake. This allows us to maintain detailed incident timelines by correlating change events with other observability data, providing valuable context during incident investigations.

#### Common Mistakes/Follow-Ups
Inadequate Event Capture: Failing to capture all relevant events can lead to incomplete incident timelines.
Delayed Ingestion: Real-time or near-real-time ingestion is crucial; delays in capturing change events can impact the accuracy of the data lake.
Data Format Issues: Poorly formatted event data can hinder analysis and querying.
Lack of Metadata: Failing to enrich data with relevant metadata (e.g., timestamps, user details) can make it harder to correlate changes with incidents.

**Comparison:**

| Event Type | Resource ID | User ID    |
|------------|-------------|------------|
| config_update | r-123456789 | u-987654321 |

**Code:**

```python
# Example Python code for transforming change events
from kafka import KafkaConsumer
import json

def process_change_event(event):
    # Enrich event with context and transform into a unified format
    enriched_event = {
        "timestamp": event["timestamp"],
        "event_type": event["event_type"],
        "resource_id": event["resource_id"],
        "user_id": event["user_id"]
    }
    return json.dumps(enriched_event)

consumer = KafkaConsumer('change_events', bootstrap_servers=['localhost:9092'])
for message in consumer:
    event = json.loads(message.value)
    transformed_event = process_change_event(event)
    # Store transformed_event in Data Lake
```

**Example input / output:**

```text
Input:
{
  "timestamp": "2023-10-01T14:30:00Z",
  "event_type": "config_update",
  "resource_id": "r-123456789",
  "user_id": "u-987654321"
}

Output:
{
  "timestamp": "2023-10-01T14:30:00Z",
  "event_type": "config_update",
  "resource_id": "r-123456789",
  "user_id": "u-987654321"
}
```

**Data:**

```json
{
  "event": {
    "timestamp": "2023-10-01T14:30:00Z",
    "event_type": "config_update",
    "resource_id": "r-123456789",
    "user_id": "u-987654321"
  }
}
```

**Flow:**

1. Change Event Source
2. CDC Mechanism
3. Kafka/Kinesis
4. Transformation Pipeline
5. Data Lake Storage
6. Incident Timeline Analysis
7. Evidence of Change Events

