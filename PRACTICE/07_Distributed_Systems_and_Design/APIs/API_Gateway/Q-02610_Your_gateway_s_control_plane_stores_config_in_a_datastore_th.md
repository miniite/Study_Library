---
type: "practice"
difficulty: "Expert"
---

# Your gateway's control plane stores config in a datastore that just became a scaling and reliability bottleneck as route count hit 100k; design the datastore-swap migration for the control plane with zero data-plane disruption, and explain how you validate correctness during dual-write.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Control Plane and Data Plane Design](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Routing_and_BFF_Patterns/Control_Plane_and_Data_Plane_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the bottleneck in our API Gateway's control plane, we need to migrate the configuration storage to a more scalable and reliable system without disrupting any ongoing traffic. This involves designing a seamless transition that allows us to validate both the old and new systems concurrently before fully switching over.

#### Triage

The core of the question is about migrating the config storage for our API Gateway's control plane while ensuring zero disruption in the data-plane. We need to design a strategy that supports dual-write, allowing validation of the new system against the existing one. The key assumptions are:

The current datastore is becoming a bottleneck.
We must maintain high availability during the migration.
We cannot tolerate any downtime or disruptions.

#### First Principles Solution
Identify the Current Datastore: Determine what type of database is currently used (e.g., PostgreSQL, Redis) and its limitations.
Select a New Datastore: Choose a more scalable solution that can handle 100k+ routes (e.g., a NoSQL database like Cassandra or a managed service like Amazon DynamoDB).
Dual-Write Mechanism: Implement a mechanism where the old and new datastores are written to simultaneously.
Validation Layer: Develop a validation layer that ensures the data in both stores is consistent.
Transition Strategy: Plan how to transition from the old to the new system with zero disruption.
Step-by-Step Solution

Analyze Current Datastore:

Current Datastore: The current datastore (e.g., Redis) has a limited number of routes it can handle efficiently.
Bottleneck Symptoms: Increased latency, read/write errors, and slow performance during route updates.

Choose New Datastore:

New Datastore: Select a NoSQL database like Cassandra or a managed service like DynamoDB to ensure high availability and scalability.
Features Consideration: Ensure the new datastore supports atomic operations, high write throughput, and low latency.

Implement Dual-Write Mechanism:

Asynchronous Writes: Configure both datastores so that writes are performed asynchronously to avoid blocking calls.
Replication Strategy: Set up replication between the old and new datastores to ensure consistent data across both.

Validation Layer:

Hash-Based Validation: Implement a validation mechanism where every write operation is accompanied by a hash or checksum of the updated configuration.
Periodic Checks: Periodically validate the consistency between the two stores using these hashes.

Transition Strategy:

Parallel Read-Write Mode: Operate in a parallel mode where both datastores are active and writes are performed to both.
Canary Rollout: Gradually roll out traffic to new configurations while validating their correctness against old configurations.
Full Transition: Once validation is successful, fully transition all write operations to the new datastore.

#### Closing Summary

By implementing a dual-write mechanism with a validation layer, we can ensure that the migration from the current datastore to the new one happens smoothly without any data-plane disruption. The key steps involve analyzing the current limitations, selecting an appropriate new technology, setting up parallel writes and validations, and finally transitioning fully when both systems are proven to be consistent.

#### Common Mistakes/Follow-Ups
Not Considering High Latency: Failing to account for latency differences between old and new datastores can lead to inconsistencies.
Overlooking Validation: Not having a robust validation mechanism can result in silent failures during the transition.
Insufficient Canary Testing: Failing to conduct thorough canary testing before full rollout can introduce unexpected issues.

**Comparison:**

```
Phase	Action
1	Analyze current datastore limitations.
2	Select a new scalable datastore.
3	Implement dual-write mechanism.
4	Develop validation layer.
5	Plan transition strategy with canary rollout.
```

**Code:**

```python
# Example of setting up dual-write mechanism
def write_route_to_datastores(route_config):
    # Asynchronous writes to old and new datastores
    old_db.write(route_config)
    new_db.write(route_config)

    # Validation step
    validate(route_config, old_db.get(route_config.id), new_db.get(route_config.id))
```

**Example input / output:**

```text
Input:
write_route_to_datastores(routeConfig)

Output:
- Old database updated with routeConfig
- New database updated with routeConfig
- Validation checks run and pass
```

**Data:**

```json
{
  "routeConfig": {
    "id": "12345",
    "uri": "/api/v1/resource"
  }
}
```

**Flow:**

1. Start
2. Analyze current datastore
3. Choose new data store
4. Implement dual-writemechanism
5. Develop validation layer
6. Plan transition strategy (canaryrollout)
7. Full transition to new datastore

