---
type: "practice"
difficulty: "Expert"
---

# How would you design delivery for a message-driven platform where deploying a consumer means schema evolution on an event backbone; how do producers, consumers, and the schema registry coordinate through CI/CD so no in-flight or replayed event ever fails to deserialize?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Schema Evolution in Event Driven Deploys](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/Schema_Evolution_in_Event_Driven_Deploys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a robust delivery system for a message-driven platform where deploying a consumer involves schema evolution on an event backbone, we need to ensure seamless coordination between producers, consumers, and the schema registry through CI/CD. The key is to maintain backward compatibility while allowing forward progress in schema evolution. This requires careful planning of how schemas are managed, versioned, and applied during deployments.

#### Triaging the Question

The core issue here involves ensuring that no in-flight or replayed event fails to deserialize after a schema change. This means we need to handle different versions of schemas gracefully. The interviewer is probing your understanding of schema management, versioning strategies, CI/CD processes, and how these fit into a message-driven architecture.

#### Assumptions
Event Sourcing: Events are stored as immutable records.
Backward Compatibility: Consumers should be able to handle older schemas.
Versioned Schema Registry: A central registry manages different versions of event schemas.
CI/CD Pipeline: Continuous Integration and Continuous Deployment processes are in place for deploying new consumers.

#### First-Principles Solution

Schema Versioning:

Define a versioning strategy, such as v1, v2, etc., or use semantic versioning (e.g., 1.0.0).

#### Cons
umers must handle events from all schema versions they may encounter.

Event Backward Compatibility:

Ensure new consumers can deserialize old events.
For example, a consumer expecting v2 should be able to handle v1 events without issues.

Central Schema Registry:

Maintain a central registry that stores all schema versions.

#### Cons
umers can query the registry for the latest or compatible schema version during deployment.

CI/CD Integration:

During CI, validate and update the schema in the registry if necessary.
During CD, ensure consumers are updated with the correct schema before starting to process events.

Handling In-Flight Events:

Use idempotent operations where possible to handle retries without side effects.
Implement event replay mechanisms that can safely apply updates without corrupting data.

Monitoring and Logging:

Monitor deserialization failures and log details for debugging and rollback if needed.
Detailed Solution

Schema Versioning Strategy:

Use a combination of version numbers in the schema name, e.g., EventName-v2.json.
Maintain a changelog that documents changes between versions.

Central Schema Registry:

Implement a service (e.g., schema-registry) that stores and serves schemas.

#### Cons
umers query this registry to get the latest or compatible schema version.

#### Cons
umer Deployment Process:

During CI, developers write tests to validate new consumers against older events.
In CD, update the consumer codebase with the latest schema from the registry.
Deploy the updated consumer and ensure it starts processing events immediately.

Handling In-Flight Events:

Use a message broker (e.g., Kafka) that supports in-order delivery guarantees.
Implement idempotent operations for consumers to handle retries gracefully.
Use event replay mechanisms to apply updates without corrupting data.

Monitoring and Logging:

Set up monitoring to track deserialization failures.
Log detailed error messages to help diagnose issues quickly.

#### Closing Summary

By implementing a robust schema versioning strategy, maintaining a central schema registry, and integrating this with CI/CD pipelines, we can ensure that deploying new consumers does not break existing event processing. This approach guarantees backward compatibility and handles in-flight events gracefully, ensuring the system remains resilient to changes.

#### Common Mistakes/Follow-Ups
Failing to Ensure Backward Compatibility: Not all schema changes are backward compatible, leading to deserialization failures.
Ignoring In-Flight Events: Without proper handling, existing data may be lost or corrupted during upgrades.
Overlooked Monitoring: Lack of monitoring can lead to undetected issues that cause downtime or data loss.

**Comparison:**

```
Schema Version	Event Name	Changes
v1	PurchaseEvent	Initial schema definition
v2	PurchaseEvent	Added additional fields
```

**Code:**

```python
# Example function for validating schema changes in CI
def validate_schema_change(new_schema_path):
    old_schemas = get_registered_schemas()
    for version, schema in old_schemas.items():
        if not is_backward_compatible(schema, new_schema_path):
            raise ValueError(f"New schema is not backward compatible with {version}")
```

**Example input / output:**

```text
Input (CI/CD Pipeline):
- New consumer codebase with updated schema v2.
- Old events in the event store.

Output (Deserialization):
- Consumers can handle both v1 and v2 events.
- In-flight or replayed events are processed correctly without errors.
```

**Data:**

```json
{
  "schema-registry": {
    "v1": {"EventName-v1.json"},
    "v2": {"EventName-v2.json"}
  }
}
```

**Flow:**

1. Yes
2. No
3. Schema Registry
4. Is Consumer Updated?
5. Fetch Latest Schema
6. Deserializing Events
7. Deploy New Consumer
8. Validate Schema Changes inCI/CD
9. Update Consumers with NewSchemas

