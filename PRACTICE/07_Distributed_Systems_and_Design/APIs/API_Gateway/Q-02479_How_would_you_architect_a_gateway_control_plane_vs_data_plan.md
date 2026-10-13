---
type: "practice"
difficulty: "Expert"
---

# How would you architect a gateway control plane vs data plane split so config pushes to 200 data-plane nodes are atomic-ish and a bad push can be rolled back in seconds without dropping traffic?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Control Plane and Data Plane Design](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Routing_and_BFF_Patterns/Control_Plane_and_Data_Plane_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to architect an API Gateway with a clear separation between the control plane and the data plane to ensure that configuration pushes are atomic and rollbacks can be performed quickly without causing any service disruptions. The control plane will handle updates and monitoring of the gateway configurations, while the data plane will route traffic to the underlying services.

#### Triaging the Question

The key aspects being probed here are:

Control Plane vs Data Plane Separation: Understanding how these two planes operate independently.
Atomicity in Configuration Pushes: Ensuring that updates do not fail halfway and can be fully rolled back if something goes wrong.
Rollback Mechanism Without Downtime: Designing a rollback process that does not drop traffic during the transition.

#### Assumptions
We have 200 data-plane nodes.
Each node must receive the same configuration update.
Failures should be detectable and handled promptly.
Rollbacks need to occur within seconds.

#### Solution
from First Principles
Control Plane Architecture

The control plane will manage the gateway configurations, enforce atomicity, and handle rollbacks. Key components include:

Configuration Manager: Manages the current state of the configuration.
Update Orchestrator: Coordinates updates across all data-plane nodes.
Health Checker: Monitors the health of each data-plane node.
Data Plane Architecture

The data plane will route traffic to the backend services based on the latest configurations from the control plane. Key components include:

Load Balancers/Forwarders: Route incoming requests to appropriate backend services.
Backend Services: The actual applications being served by the gateway.
Step-by-Step Solution
1. Designing the Control Plane

The control plane will follow a distributed architecture where each component can operate independently but communicates with others for coordination.

Configuration Manager
Update Orchestrator
Health Checker
Data Plane Nodes
2. Update Orchestrator

The update orchestrator will be responsible for applying updates atomically. It can use a consensus algorithm like Paxos or Raft to ensure that all nodes receive the same configuration before marking it as active.

Update Orchestrator
Propose New Config
Health Checker
Apply Config
3. Health Checker

The health checker will monitor the nodes to ensure they are in a healthy state before and after applying updates.

Health Checker
Check Node Health
Report Health Status
Update Orchestrator
4. Data Plane Nodes

Each node will receive the configuration update and apply it in a consistent manner, ensuring that all nodes are in sync.

Data Plane Node
Receive Config Update
Apply Config
Report Status to HealthChecker
Rollback Mechanism

To ensure atomicity and quick rollbacks, the control plane will:

Record Updates: Log all updates made by the update orchestrator.
Rollback Process: If a bad push is detected (e.g., by the health checker), revert to the previous configuration using the logs.
Update Orchestrator
Log Update
Health Checker
Raise Alarm
Rollback to Previous Config

#### Closing Summary
and Common Mistakes
Atomicity: Ensuring that a push is fully applied or not at all.
Rollback Mechanism: Designing a quick rollback process without dropping traffic.
Health Monitoring: Continuously monitoring the health of nodes to detect issues early.

#### Common mistakes
include failing to design a proper health checking mechanism, ignoring edge cases such as network partition failures, and neglecting the importance of logging for rollbacks.

Final Thoughts

By architecting an API Gateway with a clear separation between the control plane and data plane, we can achieve atomic configuration pushes and rapid rollbacks. This approach ensures that service disruptions are minimized during updates, providing a robust and reliable gateway solution.

**Comparison:**

| Node ID | Status |
|---------|--------|
| node1   | Healthy|
| node2   | Unhealthy|

**Code:**

```python
# Example of health checker in Python
import json

class HealthChecker:
    def __init__(self):
        self.config = {}

    def check_node_health(self, node_id, status):
        if status != 'healthy':
            # Log the issue and mark for rollback
            self.log_issue(node_id)
        else:
            self.update_config(node_id)

    def log_issue(self, node_id):
        # Record issues in a log file or database
        with open('health_log.json', 'a') as f:
            f.write(json.dumps({'node': node_id, 'status': status}) + '\n')

    def update_config(self, node_id):
        self.config[node_id] = {'status': 'healthy'}

# Example of update orchestrator in Python
import time

class UpdateOrchestrator:
    def __init__(self, health_checker):
        self.health_checker = health_checker
        self.new_config = {}

    def propose_new_config(self, config):
        self.new_config = config
        # Apply config to nodes one by one
        for node_id in self.nodes:
            if self.health_checker.check_node_health(node_id, 'healthy'):
                self.apply_config(node_id)

    def apply_config(self, node_id):
        print(f"Applying new configuration to node {node_id}")
        time.sleep(1)  # Simulate configuration application

# Example of data plane node in Python
class DataPlaneNode:
    def __init__(self, config_manager):
        self.config = {}
        self.config_manager = config_manager

    def receive_config_update(self, new_config):
        self.config = new_config
        print(f"Updated config to {self.config}")

# Example usage
if __name__ == '__main__':
    health_checker = HealthChecker()
    update_orchestrator = UpdateOrchestrator(health_checker)
    data_plane_node = DataPlaneNode(update_orchestrator)

    # Simulate a configuration push
    new_config = {'api_key': 'new_api_key'}
    update_orchestrator.propose_new_config(new_config)
```

**Example input / output:**

```text
Input:
- New configuration: {'api_key': 'new_api_key'}

Output (Dry-run):
1. UpdateOrchestrator proposes new config to all nodes.
2. HealthChecker checks node health before applying config.
3. Node2 is marked unhealthy and logs the issue.
4. UpdateOrchestrator applies config to healthy nodes.
5. Rollback initiated if issues detected.
```

**Data:**

```json
{
  "nodes": [
    {"id": "node1", "status": "healthy"},
    {"id": "node2", "status": "unhealthy"}
  ],
  "health_log": [
    {"node": "node2", "status": "unhealthy"}
  ]
}
```

**Flow:**

1. Propose New Config
2. Check Health
3. Node Healthy?
4. Apply Config
5. Raise Alarm
6. Update Node
7. Rollback to Previous Config

