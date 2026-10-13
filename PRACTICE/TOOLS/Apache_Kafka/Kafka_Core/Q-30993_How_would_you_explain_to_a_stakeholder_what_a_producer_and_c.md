---
type: "practice"
difficulty: "Basic"
---

# How would you explain to a stakeholder what a producer and consumer are using a data-ingestion example?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ When Not to Use Kafka](<../../../../TOOLS/Apache_Kafka/Kafka_Core/When_Not_to_Use_Kafka/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this explanation, we'll dive into how Kafka producers and consumers work together in a real-world data-ingestion scenario. A producer is an application that sends messages to a Kafka topic, while a consumer retrieves these messages from the same topic. Let's use a simple example of a logging system where web servers send logs to a Kafka cluster.

First, let’s break down what we need: producers are responsible for generating and sending data, typically structured in a message format like JSON or XML. Consumers then process this data for various purposes such as analytics or alerting. We'll see how these components interact within the broader context of the system architecture.

**Code:**

```
MERMAID
Copy
graph TD;
    A[Web Server] --> B{Produce Log};
    B --> C[Send to Kafka Topic];
    C --> D[Kafka Broker 1-3];
    D --> E[Store in Topic];
    F[Consumer Group] --> G{Consume Logs};
    G --> H[Process Logs for Analytics or Alerting];
```

**Example input / output:**

```

Input (Message from Web Server):

JSON
Copy
{
  "timestamp": "2023-10-05T14:30:00Z",
  "level": "INFO",
  "message": "User logged in"
}

Output (Processed by Consumer):

JSON
Copy
{
  "action": "Process log entries for user login",
  "details": {
    "timestamp": "2023-10-05T14:30:00Z",
    "message": "User logged in"
  }
}
```

**Data:**

```json
{
  "producer": {
    "name": "Web Server",
    "messages": [
      {"timestamp": "2023-10-05T14:30:00Z", "level": "INFO", "message": "User logged in"},
      {"timestamp": "2023-10-05T14:31:00Z", "level": "ERROR", "message": "Database connection failure"}
    ]
  },
  "consumer": {
    "name": "Analytics System",
    "actions": [
      {"action": "Process log entries"},
      {"action": "Generate alerts for errors"}
    ]
  }
}
```

