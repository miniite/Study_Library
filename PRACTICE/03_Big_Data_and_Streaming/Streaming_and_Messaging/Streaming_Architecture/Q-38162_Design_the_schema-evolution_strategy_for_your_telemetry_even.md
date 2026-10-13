---
type: "practice"
difficulty: "Intermediate"
---

# Design the schema-evolution strategy for your telemetry event topic so adding a new label field doesn't break the anomaly detector consuming it. What compatibility mode and registry do you use?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cardinality Isolation](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Cardinality_Isolation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a robust schema-evolution strategy for our telemetry event topic, ensuring that adding a new label field does not break the anomaly detector consuming it, we need to carefully consider the compatibility mode and registry setup in Apache Kafka. The key is to adopt backward compatibility, which allows us to safely add fields without breaking existing consumers. Additionally, using a Schema Registry will help manage schema versions effectively.

**Comparison:**

| Schema Version | Labels              |
|----------------|--------------------|
| V0             | app, environment    |
| V1 (Current)   | app, environment, new_field |

**Code:**

```python
# Example Python code for adding a new field while ensuring backward compatibility
from confluent_kafka import Producer

def produce_event_with_new_field(topic_name):
    producer = Producer({'bootstrap.servers': 'localhost:9092'})

    event = {
        "timestamp": 1687353240,  # Unix timestamp in seconds
        "value": 3.14,
        "labels": {  # Existing labels
            "app": "my-app",
            "environment": "dev"
        }
    }

    event["labels"]["new_field"] = "new_value"  # Adding new field

    producer.produce(topic_name, value=event)
    producer.flush()
```

**Example input / output:**

```

Given the input:

JSON
Copy
{
  "event": {
    "timestamp": 1687353240,
    "value": 3.14,
    "labels": {
      "app": "my-app",
      "environment": "dev"
    }
  }
}

The output should be:

JSON
Copy
{
  "event": {
    "timestamp": 1687353240,
    "value": 3.14,
    "labels": {
      "app": "my-app",
      "environment": "dev",
      "new_field": "new_value"
    }
  }
}
```

**Data:**

```json
{
  "event": {
    "timestamp": 1687353240,
    "value": 3.14,
    "labels": {
      "app": "my-app",
      "environment": "dev",
      "new_field": "new_value"
    }
  }
}
```

**Flow:**

1. Schema Registry
2. Produce Event
3. Schema Version
4. V0
5. D
6. V1
7. Version Control
8. Compatibility Checks
9. Auto-Register Schemas

