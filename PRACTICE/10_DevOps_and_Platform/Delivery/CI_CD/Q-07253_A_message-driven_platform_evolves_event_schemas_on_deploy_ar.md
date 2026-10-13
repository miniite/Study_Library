---
type: "practice"
difficulty: "Expert"
---

# A message-driven platform evolves event schemas on deploy; architect how producers, consumers, and the schema registry coordinate through CI/CD so no in-flight or replayed event ever fails to deserialize.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Schema Evolution in Event Driven Deploys](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/Schema_Evolution_in_Event_Driven_Deploys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a message-driven platform where event schemas can evolve on deployment, ensuring that both producers and consumers correctly handle new versions without disrupting existing workflows is crucial. This scenario requires an architecture that integrates CI/CD with schema management to maintain backward compatibility and smooth schema transitions. The key idea here is to establish a robust coordination mechanism between the schema registry, producers, and consumers during deployments.

#### Triage

The question revolves around designing a CI/CD pipeline for a message-driven system where event schemas can evolve dynamically. Specifically, we need to ensure that:

Schema Evolution: Producers and consumers must be aware of new schema versions.
Backward Compatibility: Existing events should not fail deserialization with the new schema version.
In-Flight Events Handling: Ensure ongoing messages are processed correctly even after a schema update.
Replay Protection: Prevent replaying old events that might conflict with the updated schema.

#### Assumptions
The platform uses a message broker for event delivery (e.g., Kafka, AMQP).
A central schema registry manages different versions of schemas.
We use a versioned event model where each event type has an associated schema version.

#### Cons
umers and producers are stateful components that can handle schema changes.
Step-by-Step Solution
Schema Registry Integration: The schema registry will store and manage multiple versions of schemas for each event type.
Versioning Strategy: Implement a semver-like system where schemas have unique IDs or version numbers.

#### Cons
umer-Specific Deserialization Logic: Consumers should implement deserialization logic that can handle multiple schema versions.
Schema Version Discovery: During deployment, producers and consumers must discover the latest schema version from the registry.
Deployment Strategy: The CI/CD pipeline will trigger a rolling update strategy to minimize downtime and ensure smooth transitions.
Detailed Steps

Schema Registry:

Use a central repository (e.g., AWS Schema Registry) to store event schemas with unique IDs or version numbers.
Ensure the schema registry supports multiple versions for each event type, allowing producers and consumers to reference specific versions.

Producers:

When deploying new events, ensure that the producer uses the latest schema version from the registry.
For backward compatibility, allow producers to generate events in older schemas if needed (e.g., legacy systems).

#### Cons
umers:

#### Cons
umers should implement a deserialization strategy that can handle multiple schema versions.
During deployment, consumers must update their logic to recognize and deserialize new schema versions without breaking existing workflows.

Schema Version Discovery:

Integrate the schema versioning information into the CI/CD pipeline metadata.
Use tags or labels in the deployment configuration to indicate which schema version should be used by producers and consumers.

Rolling Updates:

Deploy new versions of consumers first, ensuring they can handle both old and new schemas.
Gradually deploy new producers after confirming that all consumer instances are updated.

#### Example
Deployment Workflow
Schema Registry Update: A new schema version is added to the registry.
CI/CD Trigger: The CI/CD pipeline detects a new schema version and marks it as active.

#### Cons
umer Update:

#### Cons
umers check for updates and download the latest schema from the registry.

#### Cons
umers update their deserialization logic to handle new schemas.
Producer Deployment:
Producers are deployed with updated configuration to use the new schema versions.
Monitoring: Continuous monitoring ensures that no in-flight events fail due to schema mismatches.

#### Closing Summary

By integrating a robust schema registry and implementing a versioned event model, we can ensure that producers and consumers handle evolving schemas seamlessly during CI/CD deployments. This approach minimizes disruptions and ensures backward compatibility, allowing the system to adapt gracefully to changing requirements.

#### Common Mistakes/Follow-Ups
Mistake: Ignoring backward compatibility leads to failed deserialization of old events.

#### Follow-Up
Always test the deserialization logic with older schema versions during deployment.
Mistake: Failing to update consumer and producer configurations correctly can result in downtime or data loss.

#### Follow-Up
Implement a rollback strategy if issues are detected post-deployment.

**Comparison:**

```
Schema Version	Event Type	Fields
V1	purchase_event	user_id, item_id
V2	purchase_event	user_id, item_id, timestamp
```

**Code:**

```python
# Example of a simple deserialization function that handles multiple schema versions

def deserialize_event(event_data, schema_registry):
    latest_version = get_latest_schema_version(schema_registry)

    # Attempt to deserialize using the latest version
    try:
        return deserialize_with(latest_version, event_data)
    except DeserializationError:
        pass  # Fallback to older versions if needed

    for version in reversed(range(1, latest_version)):
        try:
            return deserialize_with(version, event_data)
        except DeserializationError:
            continue

def get_latest_schema_version(schema_registry):
    # API call to schema registry
    pass

def deserialize_with(version, event_data):
    # Specific deserialization logic for the given version
    pass
```

**Example input / output:**

```text
Input:
- Event Data: {"user_id": 123, "item_id": 456}
- Schema Registry: Latest version is v3

Output:
- Deserialized Event (v3): {"user_id": 123, "item_id": 456, "timestamp": null}

Process:
1. Check latest schema version (v3).
2. Try deserializing with v3.
3. Success: Return deserialized event.

Input:
- Event Data: {"user_id": 789, "item_id": 012}
- Schema Registry: Latest version is v2

Output:
- Deserialized Event (v2): {"user_id": 789, "item_id": 012}

Process:
1. Check latest schema version (v2).
2. Try deserializing with v2.
3. Success: Return deserialized event.
```

**Data:**

```json
{
  "latest_schema_version": 3,
  "event_types": {
    "purchase_event_v1": {"fields": ["user_id", "item_id"]},
    "purchase_event_v2": {"fields": ["user_id", "item_id", "timestamp"]}
  }
}
```

**Flow:**

1. Schema Registry Update
2. CI/CD Pipeline Trigger
3. Consumer Update
4. Rolling Deployment ofConsumers
5. Producer Deployment
6. Monitoring and Validation

