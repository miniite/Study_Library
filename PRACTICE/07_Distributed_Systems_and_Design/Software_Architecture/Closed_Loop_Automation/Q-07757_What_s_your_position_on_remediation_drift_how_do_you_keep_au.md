---
type: "practice"
difficulty: "Deep"
---

# What's your position on remediation drift: how do you keep automated fixes from silently diverging infrastructure from its declared source of truth?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Desired State Reconciliation](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Closed_Loop_Automation/Desired_State_Reconciliation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of remediation drift in an AIOps environment, we need to ensure that our closed-loop automation architecture remains tightly coupled with the declared source of truth (SOT) and actively mitigates any unintended changes or divergences over time. This involves a robust monitoring system, continuous validation mechanisms, and well-defined rollback strategies.

In essence, the question is probing how we can maintain the integrity and correctness of our infrastructure by ensuring that automated fixes do not silently drift away from the desired state defined in our SOT. Key assumptions include having reliable monitoring tools, clear policies for handling changes, and efficient reconciliation processes.

First-principles reasoning leads us to a layered solution:

Monitoring and Alerts: Implement comprehensive monitoring to detect any discrepancies between the current infrastructure state and the declared SOT. This includes real-time health checks and change detection.
Automated Validation: Use periodic or event-driven validation routines to verify that changes made by automation align with the desired configuration. This can be done through compliance checks, drift detection tools, or automated test suites.
Rollback Mechanisms: Design systems that can automatically revert any changes that do not conform to the SOT. This involves version control for configurations and state tracking mechanisms.
Feedback Loops: Ensure that feedback from these validation routines is integrated into the automation pipeline to prevent further drift.

To summarize, keeping automated fixes aligned with the declared source of truth requires a proactive approach involving robust monitoring, continuous validation, effective rollback strategies, and intelligent feedback loops.

#### Common mistakes
include underestimating the complexity of maintaining state consistency over time, relying solely on manual intervention for validation, or neglecting to implement adequate rollback mechanisms.

**Comparison:**

| Rule ID | Expected Port Range | Actual Port Range | Drift Status |
|---------|---------------------|-------------------|-------------|
| 1       | [80, 443]           | [80, 443]         | No          |
| 2       | [8080, 8081]        | [8080, 8085]      | Yes         |

**Code:**

```python
# Example of a simple drift detection function in Python

def detect_drift(current_state, desired_state):
    # This is a placeholder for an actual implementation that compares current and desired states
    return {
        "drift_detected": False,
        "details": {}
    }
```

**Example input / output:**

```text
Input:
Current State (current_state):
{
  "network_settings": {
    "firewall_rules": [
      { "rule_id": "1", "port_range": [80, 443] },
      { "rule_id": "2", "port_range": [8080, 8081] }
    ]
  }
}

Desired State (desired_state):
{
  "network_settings": {
    "firewall_rules": [
      { "rule_id": "1", "port_range": [80, 443] },
      { "rule_id": "2", "port_range": [8080, 8085] }
    ]
  }
}

Output:
Drift Detection Result: {
  "drift_detected": True,
  "details": {
    "firewall_rules": [
      {
        "rule_id": "2",
        "expected_port_range": "[8080, 8081]",
        "actual_port_range": "[8080, 8085]"
      }
    ]
}
```

**Data:**

```json
{
  "current_state": {
    "network_settings": {
      "firewall_rules": [
        { "rule_id": "1", "port_range": [80, 443] },
        { "rule_id": "2", "port_range": [8080, 8081] }
      ]
    }
  },
  "desired_state": {
    "network_settings": {
      "firewall_rules": [
        { "rule_id": "1", "port_range": [80, 443] },
        { "rule_id": "2", "port_range": [8080, 8085] } // note the change in port range
      ]
    }
  }
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. A[Current State] --> B{Monitor Changes};
4. B -- Normal -> C[Continue Automation];
5. B -- Drift Detected -> D[Detect and Validate Drift];
6. D --> E{Revert Change?};
7. E -- Yes -> F[Rollback Change];
8. E -- No -> G[Log Issue, Notify Team];

