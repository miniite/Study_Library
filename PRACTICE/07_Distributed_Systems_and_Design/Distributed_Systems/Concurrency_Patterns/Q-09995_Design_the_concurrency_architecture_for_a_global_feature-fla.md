---
type: "practice"
difficulty: "Expert"
---

# Design the concurrency architecture for a global feature-flag/config system where 100k+ instances must switch atomically to a new config without any instance ever reading a half-applied multi-field change, and a bad flag must be revertible in seconds. Justify your publication and rollback design.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Lock-free Read Paths](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Lock-free_Read_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In designing a concurrency architecture for a global feature-flag/config system with 100k+ instances, we need to ensure atomicity and consistency across all instances while avoiding race conditions that could lead to inconsistent states or partial updates. This design will focus on implementing a robust publication and rollback mechanism using a distributed coordination service like etcd or Raft-based systems, ensuring strong consistency guarantees.

#### Triaging the Question

The question requires us to design a system where 100k+ instances must switch atomically to a new configuration without any partial changes. Additionally, we need to ensure that if a bad flag is deployed, it can be reverted quickly and efficiently. The key aspects are:

Atomicity: Ensuring all instances see the same state at once.

#### Cons
istency: Avoiding half-applied updates or race conditions.
Rollback Mechanism: Enabling quick reversion to a previous configuration.

#### First Principles Solution
Centralized Coordination Service:
Use a distributed coordination service like etcd, Zookeeper, or a custom Raft-based system for consensus and state management.
Feature Flag Representation:
Represent each feature flag as a key-value pair in the coordination service with additional metadata fields (e.g., version).
Publication Flow:
Instances poll the coordination service periodically to fetch the latest configuration.
Implement a heartbeat mechanism where instances register their state with the service.
Rollback Mechanism:
Maintain a history of configurations in the coordination service, allowing quick reversion.
Detailed Design
Centralized Coordination Service

We will use etcd as our central coordination service due to its built-in support for distributed consensus and strong consistency guarantees.

Feature Flags
etcd
Distributed Consensus
Feature Flag Representation

Each feature flag is stored in etcd with the following structure:

key: Unique identifier for the feature.
value: JSON object representing the current state of the feature (e.g., {enabled: true, version: 1}).
Publication Flow

Instance Registration:

Instances register themselves in etcd by storing their state and heartbeat interval.

Polling for Configuration:

Instances periodically poll etcd to fetch the latest configuration.
A custom mechanism ensures that instances only read complete configurations; half-applied changes are ignored or marked as invalid.

Heartbeat Mechanism:

Instances send heartbeats to etcd at regular intervals, indicating their operational state and current configuration version.
This helps in identifying instances that might have failed or are out of sync.
Rollback Mechanism

Historical Configurations:

Maintain a history of all configurations stored as versions in etcd (e.g., {key: "flagA", value: {enabled: true, version: 1}}, {key: "flagA", value: {enabled: false, version: 2}}).

Rollback Trigger:

If a bad flag is detected during polling or reported by an instance, trigger a rollback.
The latest known good configuration (based on the heartbeat mechanism) is identified and applied.

Rollback Execution:

Instances are instructed to revert to the previous version of the feature flags stored in etcd.
Rollbacks are atomic; only complete versions can be reverted, ensuring no partial updates or race conditions.

#### Closing Summary

This design ensures that all instances see a consistent and complete configuration update. By leveraging a centralized coordination service like etcd, we achieve strong consistency guarantees and robust rollback mechanisms. The heartbeat mechanism provides a fail-safe by identifying out-of-sync instances, and historical configurations enable quick reversion to previous states if necessary.

#### Common Mistakes/Follow-Ups
Ignoring Race Conditions: Ensure that all operations are atomic, even in distributed environments.
Partial Updates: Use versioning and history management to avoid partial updates.

#### Performance Considerations
Monitor the performance of the coordination service and instances; optimize as needed.
Scalability: The design must handle 100k+ instances efficiently. Ensure that etcd is properly configured for high availability and load balancing.

**Comparison:**

| Instance State | Heartbeat Interval |
|----------------|--------------------|
| Instance A     | 10 seconds         |
| Instance B     | 5 seconds          |
| Instance C     | 30 seconds         |

**Code:**

```python
# Example heartbeat registration in Python
import time

def register_instance(client, key, version):
    client.set(f'instance_state/{key}', {'version': version})
    client.watch(f'heartbeat/{key}')

def poll_configuration(client, key):
    value = client.get(key)
    if value and 'enabled' in value:
        return value['enabled']
    return None

# Example heartbeat mechanism
def heartbeat(client, key, interval=10):
    while True:
        try:
            register_instance(client, key, 1)  # Simulate versioning
            time.sleep(interval)
        except Exception as e:
            print(f"Heartbeat failed: {e}")

# Example rollback logic
def revert_configuration(client, key, previous_version):
    current_value = client.get(key)
    if 'version' in current_value and current_value['version'] == previous_version:
        # Revert to the previous version
        client.set(key, previous_version)
```

**Example input / output:**

```text
Input (Instance Polling):
Request: GET /feature-flags/flagA

Response:
{
    "key": "flagA",
    "value": {
        "enabled": true,
        "version": 1
    }
}

Input (Rollback Trigger):
Event: Instance reports bad configuration state for flagA.

Response:
Request: PUT /feature-flags/flagA/version
Response:
{
    "key": "flagA",
    "value": {
        "enabled": false,
        "version": 0
    }
}
```

**Data:**

```yaml
# Example etcd configuration for feature flags
feature-flags.yaml:
  flagA: {enabled: true, version: 1}
  flagB: {enabled: false, version: 2}
```

**Flow:**

1. Yes
2. No
3. Instance
4. Register with etcd
5. Heartbeat
6. Poll Configuration from etcd
7. Process Configuration
8. Trigger Rollback?
9. Revert to Previous Version inetcd
10. Continue Operating

