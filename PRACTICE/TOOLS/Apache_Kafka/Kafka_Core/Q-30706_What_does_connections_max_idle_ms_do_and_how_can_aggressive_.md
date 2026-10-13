---
type: "practice"
difficulty: "Deep"
---

# What does connections.max.idle.ms do, and how can aggressive idle timeouts cause spurious reconnect storms?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Consumer Tuning per Workload](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Consumer_Tuning_per_Workload.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Connections.max.idle.ms in Apache Kafka is a configuration that sets the maximum allowed time interval after which an inactive connection to a broker will be closed by the broker itself. This parameter helps manage resources efficiently and prevent connections from hanging indefinitely, especially when clients are not sending or receiving messages.

The question asks about how aggressive idle timeouts (i.e., setting this value too low) can lead to spurious reconnect storms. Let's break down the core idea and explore it step by step.

**Code:**

```
MERMAID
Copy
graph TD;
    A[Connection Established] --> B[Idle Timeout Reached];
    B --> C[Broker Closes Connection];
    C --> D[Client Detects Disconnection];
    D --> E[Client Reconnects to Broker];
```

**Example input / output:**

```text
- **Input Scenario:** A client connects to a Kafka broker and remains idle for 5 minutes, while the `connections.max.idle.ms` is set to 30 seconds.
- **Output/Response:** The broker closes the connection after 30 seconds. The client detects the disconnection and attempts to reconnect immediately.
```

**Flow:**

1. Yes
2. No
3. Client Connects
4. Idle for 5 Minutes
5. C
6. Broker Closes Connection
7. Connection Remains Open
8. Client Detects Disconnection
9. Client Reconnects to Broker

