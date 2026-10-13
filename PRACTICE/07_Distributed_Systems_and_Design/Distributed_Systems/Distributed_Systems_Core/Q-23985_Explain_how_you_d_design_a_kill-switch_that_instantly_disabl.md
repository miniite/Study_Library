---
type: "practice"
difficulty: "Expert"
---

# Explain how you'd design a kill-switch that instantly disables a misbehaving auto-remediation subsystem during an incident, and why the kill-switch must not depend on the observability or control plane it might need to disable. What's your break-glass path when the AIOps automation is itself the problem?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Metastable Failures](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Retry_Budgets_and_Backoff/Metastable_Failures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

In designing a kill-switch for an auto-remediation subsystem, we prioritize decoupling from external dependencies such as observability or control planes to ensure reliable emergency shutdowns during incidents. The key is a robust, distributed design that leverages inherent reliability principles and a separate, fail-safe mechanism to bypass potential bottlenecks.

#### Triaging the Question

The core of this question revolves around creating an emergency override for a misbehaving auto-remediation subsystem. This subsystem might be part of a larger AIOps (Artificial Intelligence for Operations) framework designed to automatically detect and correct issues. However, in cases where the system itself malfunctions, we need a mechanism that can instantly disable it without relying on potentially compromised components like observability or control planes.

Designing the Kill-Switch

#### Assumptions
The auto-remediation subsystem is part of a distributed architecture.
We have access to multiple nodes or instances where the subsystem could be running.
There are external dependencies that we cannot guarantee will remain functional during an incident (observability, control planes).
Step-by-Step Solution

Identify Critical Components:

The auto-remediation subsystem should ideally be split into critical and non-critical components.
Critical components are those that must be shut down immediately if they malfunction.

Deploy Independent Kill-Switch Nodes:

Deploy separate nodes (or instances) dedicated solely to the kill-switch mechanism.
These nodes should not rely on any of the observability or control plane systems for functionality.

Design a Distributed Kill-Switch Logic:

The kill-switch nodes should implement a distributed consensus algorithm like Raft or Paxos to ensure that at least one node can agree and act upon a kill command.
Use gossip protocols (e.g., Gossip Protocol by Kademlia) for lightweight, resilient communication between the kill-switch nodes.

Implement External Fail-Safe Mechanisms:

Integrate hardware-based fail-safes such as physical switches or circuit breakers that can be manually operated in emergencies.
Implement software-based self-destruct mechanisms (e.g., time-bomb techniques) where certain conditions trigger a shutdown without relying on external systems.

Ensure Non-Blocking Communication:

Use event-driven architectures to ensure the kill-switch nodes do not block or stall critical operations.
Ensure that communication between the kill-switch and auto-remediation subsystems is non-blocking, using asynchronous messaging patterns (e.g., Kafka or MQTT).

#### Example
Architecture
Trigger
Trigger
Trigger
Manual Override
Emergency Signal
Auto-Remediation Subsystem
Kill-Switch Node 1
Kill-Switch Node 2
Kill-Switch Node 3
Control Plane / Observability
Control Plane / Observability
Control Plane / Observability
Fall-back Mechanism
Physical Switch/Circuit Breaker
Time-Bomb Mechanism

#### Closing Summary

By designing a distributed kill-switch that operates independently of the observability or control plane, we ensure that critical systems can be shut down quickly and reliably even in the face of malfunctions. The key is to leverage built-in reliability principles, distribute decision-making, and have robust fail-safe mechanisms.

#### Common Mistakes/Follow-Ups
Mistake 1: Relying on observability or control plane systems for kill-switch activation.
Mistake 2: Ignoring hardware-based fall-back mechanisms in favor of purely software solutions.

#### Follow-Up
Question: How would you test and validate the kill-switch mechanism?

**Code:**

```python
import time

def emergency_kill():
    print("Emergency Kill Signal Received")
    # Perform necessary shutdown actions here
    # For example, terminate subprocesses or send SIGKILL to processes

# Example usage
emergency_kill()
```

**Flow:**

1. Auto-Remediation Subsystem
2. Trigger Event
3. Kill-Switch Node 1
4. Kill-Switch Node 2
5. Kill-Switch Node 3
6. Control Plane / Observability
7. Control Plane / Observability
8. Control Plane / Observability
9. Fall-back Mechanism
10. Manual Override (PhysicalSwitch/Circuit Breaker)
11. Emergency Signal
12. Time-Bomb Mechanism

