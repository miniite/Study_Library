---
type: "practice"
difficulty: "Deep"
---

# How does the Schema Registry's subject naming strategy (topic vs record vs topic-record) affect a topic that legitimately carries multiple event types for a data platform?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Schema Formats and Registry](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Schema_Formats_and_Registry.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with topics in Apache Kafka, particularly within a data platform that supports multiple event types, the choice of subject naming strategy—whether it’s topic, record, or topic-record—can significantly impact the design and maintenance of your schema registry. Here, we'll delve into how each strategy works and its implications on topic management.

#### Triage

The question is asking about the differences in how Kafka's Schema Registry handles subject naming strategies for a single topic that needs to support multiple event types. The interviewer likely wants to probe your understanding of how these strategies affect schema registration, versioning, and eventual consumption by downstream systems.

#### First-Principles Solution
Topic Strategy:
In this approach, the topic is simply named based on a business context or the type of events it carries.

#### Example
user.events. This means all event types related to users are published to this single topic.
Record Strategy:
Each event type has its own schema and publishes directly to its corresponding topic, but with the same base name (e.g., user).

#### Example
user.create, user.update. The schema registry maintains a separate record for each type of event under user.
Topic-Record Strategy:
Combines elements from both previous strategies; it names topics based on their specific use case and has unique schemas registered.

#### Example
user.events.create for create events, user.events.update for update events.
Detailed Solution
Topic Strategy

#### Pros
Simplified topic management. Easier to manage fewer topics.
E.g., if you have a single topic called sensor.readings, it can handle various types of sensor data without additional complexity.

#### Cons
Schema versioning and management become more complex as the number of event types grows, leading to potential schema conflicts or confusion.

#### Cons
uming clients need to understand which part of the topic payload corresponds to each event type.
Record Strategy

#### Pros
Each event type has its own schema, making it easier to manage versions independently.
E.g., if user.create and user.update are managed separately, changes can be applied without affecting the other types.

#### Cons
More topics mean increased complexity in topic management and possibly more overhead during deployment.
Increased namespace pollution as each event type gets its own topic.
Topic-Record Strategy

#### Pros
Balances simplicity with versioning. It leverages the benefits of both strategies.
E.g., user.events.create and user.events.update have their own schemas but are grouped under a broader user.events namespace, making it easier to manage related events together.

#### Cons
More complex schema management as you need to define multiple topic names and their associated schemas.

#### Closing Summary

Choosing the right subject naming strategy depends on your specific use case. For platforms with many event types, using a topic-record approach can be beneficial due to its balance between simplicity and manageability. However, for simpler scenarios where fewer event types are involved, sticking with just topics might suffice.

#### Common Mistakes/Follow-Ups
Mistake: Overlooking the impact of schema versioning in complex schemas.
Always consider how schema changes will affect existing consumers.

#### Follow-Up
Discuss how confluent-specific tools like Confluent CLI or Schema Registry UI can help manage these strategies.

#### Example
Input/Output for Schema Registration
JSON
// Example topic registration for "user.events.create" using the Record Strategy
{
  "name": "user",
  "version": 1,
  "payloadschema": {
    "$id": "http://example.com/root.json",
    "type": "object",
    "title": "The Root Schema",
    "description": "The root schema comprises the entire JSON document.",
    ...
}

#### Example
Mermaid Flowchart
No
Yes
No
Yes
Topic Name
Is it a Record Strategy?
Use Topic-Record Naming
Register Records Individually
Simplified SchemaManagement
Increased Topic Complexity
Versioning Flexibility
Topic Naming
Simplify Management?
Maintain All Events in a SingleTopic
Use Topic-Record Strategy forBetter Organization

**Comparison:**

```
Schema Registry Strategy	Pros	Cons
Topic	Simplified topic management. Easier to manage fewer topics.	Schema versioning and management become more complex as the number of event types grows, leading to potential schema conflicts or confusion. Consuming clients need to understand which part of the topic payload corresponds to each event type.
Record	Each event type has its own schema, making it easier to manage versions independently.	More topics mean increased complexity in topic management and possibly more overhead during deployment. Increased namespace pollution as each event type gets its own topic.
Topic-Record	Balances simplicity with versioning. Easier to manage related events together while still allowing for independent schema changes.	More complex schema management as you need to define multiple topic names and their associated schemas.
```

**Code:**

```python
# Example Python code to register schemas using Confluent's Schema Registry REST API

import requests

def register_schema(topic_name, schema_json):
    url = "http://localhost:8081/subjects/{}/versions".format(topic_name)
    headers = {"Content-Type": "application/vnd.schemaregistry.v1+json"}

    response = requests.post(url, data=schema_json, headers=headers)
    if response.status_code == 200:
        print("Schema registered successfully")
    else:
        print("Failed to register schema: ", response.content)

# Example JSON payload
schema_payload = {
    "name": "user",
    "version": 1,
    "subjectName": "user.events.create",
    "schema": "{\"type\": \"record\", \"name\": \"UserCreateEvent\"}"
}

register_schema("user", schema_payload)
```

**Example input / output:**

```text
// Example of registering a user creation event schema

Input:
{
    "name": "user",
    "version": 1,
    "subjectName": "user.events.create",
    "schema": "{\"type\": \"record\", \"name\": \"UserCreateEvent\"}"
}

Output:
Schema registered successfully
```

**Data:**

```json
{
  "name": "sensor.readings",
  "version": 1,
  "subjectName": "sensor.events.create",
  "schema": "{\"type\": \"record\", \"name\": \"SensorReadingEvent\"}"
}
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. Topic Name
6. Is it a Record Strategy?
7. Use Topic-Record Naming
8. Register Records Individually
9. Simplified SchemaManagement
10. Increased Topic Complexity
11. Versioning Flexibility
12. Topic Naming
13. Simplify Management?
14. Maintain All Events in a SingleTopic
15. Use Topic-Record Strategy forBetter Organization

