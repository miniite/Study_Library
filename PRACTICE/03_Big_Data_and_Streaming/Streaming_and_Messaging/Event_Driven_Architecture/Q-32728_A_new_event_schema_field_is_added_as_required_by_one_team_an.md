---
type: "practice"
difficulty: "Deep"
---

# A new event schema field is added as required by one team and it silently breaks another team's consumer in production. How do you set up the org and tooling so this can't happen again?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Breaking Schema Changes](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Fat_vs_Thin_Events/Breaking_Schema_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of breaking consumers due to schema changes, we need a robust approach that combines strong governance, clear communication, and effective tooling. The key is to ensure that all changes to event schemas are managed through formal processes, validated by automated tools, and communicated transparently across teams.

The problem stems from insufficient coordination between different teams using message queues in an event-driven architecture. To prevent such issues, we need to establish a comprehensive schema management process and leverage tooling for validation and enforcement.

First, let's break down the question: The core challenge is managing schema changes without breaking existing consumers. We need to ensure that any new field additions or modifications are systematically handled and validated before deployment. This involves setting up governance practices, using schema versioning tools, and maintaining clear communication channels.

#### Assumptions

Teams are working on different services consuming events from the same message queue.
Event schemas are stored in a central repository (e.g., JSON schema registry).

#### Cons
umers use a standard framework or library for reading and processing events.

Now, let's reason step-by-step:

Step 1: Centralized Schema Management

Centralize all event schemas in a reliable, version-controlled repository like AWS Schemas, Google Cloud Pub/Sub Schemas, or an open-source solution like Confluent Schema Registry. This ensures that every schema is versioned and documented.

Step 2: Automated Validation Tools

Implement tools to validate incoming events against the current schema versions. For example:

Use a middleware component in your event processing pipeline to check if each message adheres to the latest schema.
Integrate static analysis tools that can lint messages before they reach consumers.

#### Example
code for validation could look like this:

PYTHON
def validate_event(event, schema):
    # Implement validation logic using a library like jsonschema
    return jsonschema.validate(event, schema)
Step 3: Versioning and Backwards Compatibility

Adopt a versioned schema approach where each new field addition or modification increments the schema version. Consumers must explicitly declare which versions they support to avoid breaking changes.

#### Example
schema versioning:

JSON
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Event V123",
  "type": "object",
  "properties": {
    // existing properties
    "new_field_name": { "type": "string" }
  },
  "required": ["new_field_name"]
}
Step 4: Clear Communication and Change Management

Establish a change management process where schema changes are proposed, reviewed, and approved by relevant stakeholders. Use pull requests or similar workflows to ensure all teams are informed about upcoming changes.

#### Example
PR workflow:

Developer proposes a new field in the schema.
Schema changes are reviewed for backwards compatibility.
Changes are merged with tests passing.
Documentation is updated.

#### Cons
umers are notified of the new schema version and required changes.
Step 5: Monitoring and Alerts

Set up monitoring and alerting systems to detect any schema validation failures in production. This ensures that issues are caught early and resolved before they impact multiple consumers.

#### Example
alert configuration:

YAML
triggers:
- event_validation_error_count: "10"
actions:
- type: notify
  level: critical
  channels: slack

#### Closing Summary

By implementing a structured approach to schema management, including centralized versioning, automated validation, clear communication, and robust monitoring, we can prevent breaking changes from affecting consumers in production. This not only ensures stability but also promotes better collaboration across teams.

#### Common mistakes
include:

Failing to enforce schema versions.
Ignoring existing consumer dependencies when making changes.
Not having a formal change management process.

**Comparison:**

```
Schema Version	Fields
V100	username
	timestamp
V123	new_field_name
	...
```

**Code:**

```python
def validate_event(event, schema):
    # Implement validation logic using a library like jsonschema
    return jsonschema.validate(event, schema)
```

**Example input / output:**

```

Example input:

JSON
Copy
{
  "event_type": "user_login",
  "username": "john_doe",
  "timestamp": "2023-10-01T14:30:00Z"
}

Example output:

JSON
Copy
{
  "$schema_version": "V123",
  "event_type": "user_login",
  "username": "john_doe",
  "timestamp": "2023-10-01T14:30:00Z"
}
```

**Data:**

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Event V123",
  "type": "object",
  "properties": {
    // existing properties
    "new_field_name": { "type": "string" }
  },
  "required": ["new_field_name"]
}
```

**Flow:**

1. Yes
2. No
3. Schema Change Request
4. Propose Changes
5. Review & Validate
6. Merge Changes
7. Retry or Re-work
8. Document Changes
9. Notify Consumers
10. Monitor for Issues

