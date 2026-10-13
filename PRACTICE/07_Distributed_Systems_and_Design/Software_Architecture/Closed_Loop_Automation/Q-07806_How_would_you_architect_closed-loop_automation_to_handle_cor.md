---
type: "practice"
difficulty: "Expert"
---

# How would you architect closed-loop automation to handle correlated failures across the dependency graph so it fixes the cause, not every downstream symptom?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Business Aware Automation](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Closed_Loop_Automation/Business_Aware_Automation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR Summary:

To architect a robust closed-loop automation system that handles correlated failures in a dependency graph, we need to focus on three key areas: failure detection, root cause analysis (RCA), and automated resolution. The solution involves leveraging a dynamic dependency graph, machine learning for pattern recognition, and a resilient failure handling mechanism.

#### Triaging the Question

The question is asking how to design an automation system that can identify and fix the root cause of correlated failures rather than just addressing each downstream symptom. This requires understanding dependencies between services/components, detecting failures accurately, analyzing root causes, and automating corrective actions. The interviewer may be probing your knowledge in fault tolerance, dependency management, machine learning models for pattern recognition, and automated decision-making.

#### First-Principles Solution
Dependency Graph: Construct a dynamic graph representing the dependencies between services/components. This helps in understanding how failures propagate through the system.
Failure Detection: Implement a monitoring system that can detect failure events across all components.
Root Cause Analysis (RCA): Use machine learning to identify patterns and correlate symptoms with potential root causes.
Automated Resolution: Develop algorithms to determine the best corrective action based on RCA findings.
Step-by-Step Solution:

Dependency Graph Construction:

Create a graph where nodes represent services/components, and edges indicate dependencies between them.
Update this graph dynamically as new components are added or existing ones change their behavior.

Failure Detection:

Monitor each node in the dependency graph for failure conditions using appropriate health checks (e.g., response times, error rates).
Use event-driven architectures to trigger alerts when a failure is detected.

Root Cause Analysis (RCA):

Implement a machine learning model that learns from historical failure data.
Train the model on various features such as time of day, service load, and correlation metrics between components.
When a failure occurs, feed the current state into the ML model to predict potential root causes.

Automated Resolution:

Based on the predicted root cause, design algorithms that can automatically apply corrective actions (e.g., restarting services, rerouting traffic).
Ensure these actions are safe and do not exacerbate the situation by implementing rollback mechanisms.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

The key to successful closed-loop automation is ensuring that you handle failures at their source rather than just treating symptoms. This involves a comprehensive approach including dynamic dependency mapping, real-time failure detection, advanced root cause analysis, and intelligent automated resolution. Common pitfalls include over-reliance on heuristics, ignoring edge cases, and not adequately testing rollback mechanisms.

#### Follow-Up

How do you handle false positives in the RCA?
What strategies would you use to ensure the automated actions are safe?
Can you walk through an example of how your system would resolve a correlated failure?
Self-Check:

This solution is comprehensive and covers all critical aspects of building a robust closed-loop automation system.

**Comparison:**

| Failure | Time        | Root Cause Prediction   |
|---------|------------|------------------------|
| Node1   | 09:30 AM   | node2 service overload  |
| Node5   | 11:45 PM   | network latency issue   |

**Code:**

```python
# Example code for dynamic dependency graph construction
class DependencyNode:
    def __init__(self, service_name):
        self.service_name = service_name
        self.dependencies = []
        self.failures = 0

nodes = {service.name: DependencyNode(service.name) for service in all_services}
```

**Example input / output:**

```text
Input:
- Failure detected in node1

Output:
- Node1 is restarted.
- Alerts sent to ops team about potential root causes.
- ML model predicts likely root cause (e.g., "node2" dependency issue).
- Automated actions initiated to resolve the root cause without affecting other nodes.
```

**Data:**

```json
{
  "dependencyGraph": {
    "node1": {"dependencies": ["node2", "node3"]},
    "node2": {"dependencies": []},
    "node3": {"dependencies": ["node4"]}
  }
}
```

**Flow:**

1. Yes
2. No
3. Failure Detected
4. Is this a new failure?
5. Maintain state
6. Update Dependency Graph
7. Maintain state and monitor
8. RCA Model Prediction
9. Corrective Actions Initiated

