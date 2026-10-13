---
type: "practice"
difficulty: "Deep"
---

# How would you enforce backward and forward compatibility of event schemas in a Schema Registry so producers can't break your downstream Spark pipelines?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Schema Drift Handling](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Schema_Drift_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure backward and forward compatibility of event schemas in a Schema Registry while working with Data Pipelines (Airflow/Spark), we need to establish robust schema management practices that allow for seamless integration between producers and consumers without breaking downstream Spark pipelines. This involves designing a system where new versions of schemas can be introduced gradually, ensuring that both old and new systems can process data correctly.

Firstly, let's break down the problem: the goal is to prevent schema changes from disrupting existing workflows, which are likely built on a specific version of the schema. To achieve this, we need to manage schema evolution carefully by adhering to certain rules and leveraging tools like Apache Avro or JSON Schema for schema registration.

The key idea here is to maintain a clear understanding of how each schema change impacts the downstream systems, especially Spark pipelines. We'll start by outlining the steps required to enforce backward and forward compatibility.

**Comparison:**

```
Schema Version	Changes	Impact on Consumers
1.0	Initial schema	Not applicable
2.0	Add "userAgent" field	Optional
3.0	Change "eventType"	Must update logic
```

**Code:**

```python
# Example Python snippet to register schemas in an Avro Schema Registry
from avro.datafile import DataFileReader, DataFileWriter
from avro.io import DatumReader, DatumWriter
import avro.schema

schema_registry = 'http://localhost:8081'

def register_schema(topic_name, schema_str):
    # This function registers a new or updates an existing schema in the Avro Schema Registry
    headers = {'Content-Type': 'application/json'}
    payload = {
        "schema": schema_str,
        "subject": f"{topic_name}-value"  # Kafka subject for this topic's value schema
    }
    requests.post(f'{schema_registry}/subjects/{topic_name}-value/versions', json=payload, headers=headers)

def get_latest_schema(topic_name):
    response = requests.get(f'{schema_registry}/subjects/{topic_name}-value/latest')
    return response.json().get('schema')

# Example usage
register_schema("user_events", '{"type": "record", ... }')
latest_user_event_schema = get_latest_schema("user_events")
```

**Example input / output:**

```text
Input:
New user event schema with a new field "userAgent":
{
  "$id": "2",
  "title": "User Events Schema",
  "type": "object",
  "properties": {
    "userId": {"type": "string"},
    "eventType": {"type": "string"},
    "eventTimestamp": {"type": "integer"},
    "userAgent": {"type": "string"}
  }
}

Output:
1. Schema registered successfully.
2. Downstream Spark pipeline is updated to handle the new "userAgent" field.

Note: The Spark pipeline should use AvroIO or similar library to read the schema and process data accordingly.
```

**Data:**

```json
{
  "$id": "1",
  "title": "User Events Schema",
  "type": "object",
  "properties": {
    "userId": {"type": "string"},
    "eventType": {"type": "string"},
    "eventTimestamp": {"type": "integer"}
  }
}
```

**Flow:**

1. Producer
2. Schema Registry
3. Schema Registration
4. Subject Definition
5. Version Management
6. Consumer
7. Kafka Topic Consumption
8. Spark Pipeline Integration
9. Auditing and Monitoring

