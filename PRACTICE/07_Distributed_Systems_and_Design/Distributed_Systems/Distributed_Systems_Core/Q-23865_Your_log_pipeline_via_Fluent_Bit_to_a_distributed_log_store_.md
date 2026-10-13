---
type: "practice"
difficulty: "Intermediate"
---

# Your log pipeline via Fluent Bit to a distributed log store started dropping records during an incident spike, exactly when you needed the logs most. Walk me through where buffering versus dropping should live and how you avoid the pipeline masking the very incident it feeds detection.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Tracing](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/Distributed_Tracing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a log pipeline with Fluent Bit and a distributed log store, the challenge of maintaining data integrity during high-traffic spikes is paramount. Buffering can help manage this by temporarily storing logs before they are processed or archived. However, it's crucial to carefully place buffering within the pipeline and understand its trade-offs to ensure that the system does not mask critical information about the incident.

In this scenario, we need to address where buffering should be implemented and how to design a resilient pipeline that doesn’t drop essential records under load. Let’s break down the solution from first principles.

**Code:**

```
MERMAID
Copy
graph LR;
    A[Log Source] --> B[Fluent Bit];
    B --> C{Buffer?};
    C -- Yes --> D[Distributed Log Store (e.g., Elasticsearch, Kafka)];
    C -- No --> E[Drop Records];
```

**Example input / output:**

```text
Input:
[{"timestamp": "2023-10-01T12:00:00Z", "logLevel": "ERROR", "message": "Database connection failed", "applicationName": "db"}]

Output (with buffering):
[{"timestamp": "2023-10-01T12:00:05Z", "logLevel": "ERROR", "message": "Database connection failed", "applicationName": "db"}]
```

**Data:**

```text
# Sample Data Schema
{
  "timestamp": "2023-10-01T12:00:00Z",
  "logLevel": "INFO",
  "message": "System is under heavy load",
  "applicationName": "webapp"
}
```

