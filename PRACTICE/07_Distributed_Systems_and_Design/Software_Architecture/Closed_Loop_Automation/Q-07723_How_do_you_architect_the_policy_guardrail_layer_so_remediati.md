---
type: "practice"
difficulty: "Expert"
---

# How do you architect the policy/guardrail layer so remediation actions are governed centrally but authored and owned by individual service teams?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Safe Closed-loop Remediation](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Closed_Loop_Automation/Safe_Closed-loop_Remediation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a closed-loop automation architecture where remediation actions are governed centrally but authored and owned by individual service teams, we need to strike a balance between centralized control and decentralized autonomy. This involves designing a robust policy/guardrail layer that enforces consistency while allowing flexibility for different teams. The solution will involve several key components: a central governance engine, service team-defined policies, and an execution framework.

Firstly, the central governance engine acts as the authoritative source of truth for guardrails and remediation policies. It ensures compliance across all services by defining and enforcing rules that are critical to maintain system integrity and security. This engine must be highly scalable, resilient, and capable of integrating with various monitoring and alerting systems to identify violations.

Secondly, individual service teams own the specific policies related to their domain. Each team will define and manage its own set of remediation actions based on best practices and unique requirements. This decentralization fosters innovation and ensures that local context is considered in policy formulation.

Thirdly, an execution framework is necessary to translate these policies into actionable steps. This could involve integrating with existing automation tools or developing custom scripts. The framework should be modular, allowing easy integration of different remediation actions while ensuring they are executed securely.

**Comparison:**

```
Policy	Service	Condition	Action
High CPU Usage Alert	web	cpu_usage > 80%	restart_service
Disk Space Low Alert	db	disk_space < 10%	shutdown_db
```

**Code:**

```python
# Example of a policy definition in YAML format
policy:
  service: web
  name: High CPU Usage Alert
  condition: cpu_usage > 80%
  action: restart_service

# Example of an execution framework snippet
def execute_action(action):
    if action == "restart_service":
        # Code to restart the web server
        pass
```

**Example input / output:**

```text
Input (sample policy):
{
  "policies": [
    {
      "service": "web",
      "name": "High CPU Usage Alert",
      "condition": "cpu_usage > 80%",
      "action": "restart_service"
    }
  ]
}

Output (execution plan):
Action: restart_service
Target: web service
```

**Data:**

```json
{
  "policies": [
    {
      "service": "web",
      "name": "High CPU Usage Alert",
      "condition": "cpu_usage > 80%",
      "action": "restart_service"
    },
    {
      "service": "db",
      "name": "Disk Space Low Alert",
      "condition": "disk_space < 10%",
      "action": "shutdown_db"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Central Governance Engine
4. Is the violation severe?
5. Execute Central Policy
6. Forward to Service Team
7. Service Team Defines LocalRemediation
8. Central Policy Enforcement
9. Monitor & Alert System

