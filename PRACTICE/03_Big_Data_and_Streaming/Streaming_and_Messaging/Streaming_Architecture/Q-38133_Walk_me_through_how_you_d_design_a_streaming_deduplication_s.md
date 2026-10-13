---
type: "practice"
difficulty: "Deep"
---

# Walk me through how you'd design a streaming deduplication stage for alerts so the same underlying fault firing across 200 hosts collapses into one incident. What state and what window do you use?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Duplicates During Rebalance](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Duplicates_During_Rebalance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a robust real-time streaming deduplication stage for alerting, we need to ensure that duplicate alerts from multiple sources are collapsed into a single incident when they represent the same underlying fault. This involves understanding the nature of alerts and implementing an appropriate state management strategy with a suitable time window.

The core idea is to use a combination of message attributes (such as host ID, timestamp, and alert payload) to identify duplicates within a specific temporal window. This approach helps in maintaining high accuracy while ensuring minimal delay in processing critical alerts.

Firstly, we need to define the state and window:

State: We maintain an in-memory or distributed data store that tracks unique alerts.
Window: The time frame during which we consider alerts as potential duplicates (e.g., 5 minutes).

Next, let's break down the solution step by step:

Data Ingestion:

Alerts are ingested from various sources (200 hosts) in real-time via a message broker like Kafka.

Key Extraction:

Each alert is transformed to extract critical components that form its unique identifier, such as host_id, timestamp, and alert_payload. For example, if two alerts have the same host_id and timestamp but different alert_payload, they are considered distinct.

State Management:

We maintain a state store (e.g., Redis for in-memory, or a distributed database like Cassandra) that maps unique keys to alert details.
For each incoming alert, we compute its key using the extracted components and check if it exists in our state store.
PYTHON
def get_alert_key(host_id, timestamp, payload):
    return f"{host_id}_{timestamp}_{payload}"

Duplicate Detection:

If the key does not exist, we add the alert to the state store along with relevant details (e.g., incident_id).
If the key exists, it means this is a duplicate alert, and we increment an incident counter or update existing incident details.

Window Consideration:

Alerts within our defined time window are considered potential duplicates.
We can implement a sliding window mechanism to ensure alerts from before the current timestamp but within the window are also checked.

Incident Collapsing:

When an alert is confirmed as a duplicate, we collapse it into the existing incident by updating its details or aggregating metrics (e.g., increasing severity if multiple alerts indicate the same issue).

Notification and Action:

Once a unique incident is identified, it triggers a notification to relevant stakeholders via a monitoring system.

Now, let's summarize the key components and potential pitfalls:

State Store: Efficiently handles state updates and queries with low latency.
Window Management: Ensures timely detection of duplicates by considering recent events.
Key Extraction: Robust enough to differentiate between similar but distinct alerts.

#### Common mistakes
include:

Ignoring edge cases where different hosts might report the same issue at slightly different times.
Failing to properly handle state updates and concurrency issues in a distributed system.
Not adequately considering the trade-off between strictness of deduplication rules and the risk of missing critical alerts.

