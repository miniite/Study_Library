---
type: "practice"
difficulty: "Deep"
---

# You promote a model, but half the fleet picks up the new version and half is stale for several minutes because of caching and staggered rollout. Explain the mixed-version risk (inconsistent predictions to the same user) and how you'd make the switch atomic.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Mixed Version Fleets](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When rolling out a new version of an ML model in a distributed system, it's critical to ensure consistency and avoid mixed-version scenarios where different instances return conflicting predictions for the same user. This can lead to inconsistent experiences or incorrect decisions, which is unacceptable in many applications, especially those involving financial transactions or healthcare diagnostics.

The challenge here lies in managing the rollout process to make sure all instances of the model are updated atomically. Here’s a step-by-step approach:

Understand the Current State: The current system has a caching layer and a staggered rollout mechanism, leading to half the fleet using the new version while the other half is still using the old one.

Identify Risks: Mixed-version usage can cause issues like:

Inconsistent predictions for the same user across different instances.
Potential loss of user trust due to unpredictable outcomes.

Plan the Switch: To address this, we need a way to synchronize the model updates across all nodes in the fleet atomically.

#### Solution
Approach:

Centralized Control: Introduce a central service that coordinates the update process.
Update Flag: Use an atomic flag or lock mechanism within each node to signal when it’s safe to update the model.
Rollback Mechanism: Ensure there is a fail-safe rollback plan in case of issues during the update.

Here's a more detailed breakdown:

Current Setup:

Model instances are deployed across multiple nodes, with some caching layers.
Caching and staggered rollout mechanisms ensure gradual updates to avoid sudden failures.

Identifying Risks:

User predictions can be inconsistent if the model version differs between nodes.
This inconsistency can lead to erroneous decisions or user frustration.

#### Solution
Design:

Central Coordinator: Use a central service (e.g., Zookeeper, Consul) to manage the update process.
Atomic Flagging: Implement an atomic flag mechanism where each node checks with the coordinator before updating its model.
Rollback Strategy: Have a strategy in place to revert nodes back to the old version if issues are detected.

Step-by-Step Process:

The central service marks all nodes as "upgrading" and waits until it receives confirmation from at least half of them that they have started the update.
Once the threshold is met, the coordinator signals a full update across all remaining nodes simultaneously.
After the final node confirms the update, the process is marked complete.

#### Example
Workflow:

CODE
1. Central Service: Set "Upgrade Flag" to True
2. Nodes Check Flag: If True, Proceed with Update (atomic operation)
3. Central Service Confirms Majority of Nodes Upgraded
4. All Nodes Update Model Simultaneously
5. Central Service Sets "Upgrade Flag" Back to False

Risk Mitigation:

Use versioning and metadata to track model states.
Monitor for any anomalies or issues during the update.

#### Closing Summary
By implementing a centralized coordination strategy with atomic flagging, we can ensure that all instances of the ML model are updated atomically, reducing mixed-version risks and maintaining consistency across the fleet.

