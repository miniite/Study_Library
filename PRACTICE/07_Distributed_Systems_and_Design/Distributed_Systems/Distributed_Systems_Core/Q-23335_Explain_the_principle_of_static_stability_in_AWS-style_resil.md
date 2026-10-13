---
type: "practice"
difficulty: "Expert"
---

# Explain the principle of 'static stability' in AWS-style resilient design: the data plane keeps working even when the control plane is down. Design a system where failover, service discovery, and scaling don't depend on a live control plane during the very outage that broke it.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Static Stability](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Static_Stability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Static stability in AWS-style resilient designs ensures that critical services continue to function even when the central control plane is unavailable. This concept is crucial for designing distributed systems where the data plane's operations are decoupled from the control plane, enabling a graceful degradation of service without losing all functionality.

In essence, static stability means maintaining system reliability through local decision-making and pre-provisioned fallback mechanisms. The key idea is that when the control plane fails, nodes in the network can continue to operate based on static configurations or cached data, ensuring minimal disruption.

To achieve this, we need a system where failover, service discovery, and scaling are independent of live control-plane updates during outages. Let's break down the components step-by-step:

Pre-provisioned Service Discovery: Use static configuration files or local caches to store service information.
Local Decision-Making: Implement self-healing mechanisms that can make decisions based on local state and pre-defined rules.
Decoupled Scaling Mechanisms: Design auto-scaling policies that are not reliant on real-time control-plane updates.

Let's dive deeper into each component:

Pre-provisioned Service Discovery

In a resilient design, services should be aware of their peers through static configuration files or local caches. This ensures that nodes can discover and communicate with other services even if the central control plane is down. For example, DNS records could point to pre-known IP addresses or service identifiers.

Service A
Service B
Local Decision-Making

Nodes should be able to make decisions based on their local state and predefined rules. This can include mechanisms like leader election algorithms, gossip protocols for health checks, and fallback strategies in case of failures.

For instance, a node could use a simple majority voting mechanism to decide if it's healthy or needs to take over responsibilities from another node. Here’s an example pseudo-code:

PYTHON
class Node:
    def __init__(self):
        self.is_leader = False

    def check_health(self):
        # Check health logic here
        return True  # Assume healthy for now

    def decide_leadership(self):
        if self.check_health():
            self.is_leader = True
Decoupled Scaling Mechanisms

Scaling policies should be pre-defined and can be triggered based on local metrics. These policies could include simple rules like "scale up by one instance when CPU utilization exceeds 80%." Pre-defined scaling groups or auto-scaling groups with predefined policies ensure that nodes can scale independently without relying on live control-plane updates.

For example, an auto-scaling group could have a policy to add one more node if the average CPU load of all nodes in the group is greater than 75%.

PYTHON
class AutoScalingGroup:
    def __init__(self, max_instances):
        self.max_instances = max_instances

    def scale_out(self):
        # Logic to add an instance based on criteria
        pass

#### Closing Summary
and Common Mistakes/Follow-Ups

Static stability is a critical principle for building resilient distributed systems. By decoupling the data plane from the control plane, we can ensure that services remain functional during outages. However, some common mistakes include:

Over-reliance on Control Plane: Failing to pre-provision fallback mechanisms.
Inadequate Local Decision-Making: Not implementing self-healing and decision-making at the node level.
Ignoring Edge Cases: Failing to consider scenarios like multiple simultaneous outages or degraded network conditions.

When designing such systems, it’s important to continuously test and validate these static stability strategies to ensure they hold up in real-world scenarios.

**Comparison:**

```
Node ID	IP	Health
1	10.0.0.1	True
2	10.0.0.2	False
Service	Nodes
A	[10.0.0.1]
B	[10.0.0.2]
```

**Code:**

```python
class Node:
    def __init__(self):
        self.is_leader = False

    def check_health(self):
        # Check health logic here
        return True  # Assume healthy for now

    def decide_leadership(self):
        if self.check_health():
            self.is_leader = True

class AutoScalingGroup:
    def __init__(self, max_instances):
        self.max_instances = max_instances

    def scale_out(self):
        # Logic to add an instance based on criteria
        pass
```

**Example input / output:**

```text
Input:
- Service A requests connection to Service B.

Output:
- Node with IP 10.0.0.1 (Service A) connects to node with IP 10.0.0.2 (Service B) using static DNS records.
```

**Data:**

```json
{
  "nodes": [
    {"id": 1, "ip": "10.0.0.1", "health": true},
    {"id": 2, "ip": "10.0.0.2", "health": false}
  ],
  "services": {
    "serviceA": ["10.0.0.1"],
    "serviceB": ["10.0.0.2"]
  }
}
```

**Flow:**

1. Node checks health
2. Is node healthy?
3. Node declares leadership ifhealthy
4. Service requests connection toanother service
5. Static DNS records resolvetarget IP
6. Connection established

