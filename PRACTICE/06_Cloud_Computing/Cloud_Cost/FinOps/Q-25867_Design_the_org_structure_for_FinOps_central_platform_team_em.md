---
type: "practice"
difficulty: "Expert"
---

# Design the org structure for FinOps: central platform team, embedded champions, or federated ownership? Argue for a model given a 500-team engineering org and explain the accountability contract each team signs up to.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Scaling Cost Governance](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Scaling_Cost_Governance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a large engineering organization with over 500 teams, designing an effective FinOps framework is critical for managing cloud costs and ensuring financial governance. The three common models—central platform team, embedded champions, or federated ownership—each have unique strengths and trade-offs. For a 500-team org, the federated ownership model often strikes a balance between central oversight and local flexibility, but it's crucial to formalize an accountability contract that each team signs up to.

**Comparison:**

| Team Model         | Central Platform Team              | Embedded Champions             | Federated Ownership           |
|--------------------|------------------------------------|-------------------------------|------------------------------|
| **Centralization** | High (all control)                 | Moderate (some control)        | Low (minimal direct oversight) |
| **Local Flexibility** | Low                               | Moderate                       | High                          |
| **Communication & Coordination** | High                              | Moderate                       | Moderate                      |
| **Scalability**     | Limited                             | Moderate                        | High                          |
| **Accountability**  | Clear but may be disjointed        | Clear and localized            | Clear, self-driven            |

**Code:**

```text
# Example Accountability Contract
class FinOpsAccountabilityContract:
    def __init__(self, team_name):
        self.team_name = team_name
        self.cost_savings_goals = 0.0
        self.cloud_utilization_metrics = []

    def set_cost_savings_goal(self, percentage):
        self.cost_savings_goals = percentage

    def report_metrics(self, metrics_data):
        self.cloud_utilization_metrics.append(metrics_data)
```

**Example input / output:**

```text
# Example input and output for the Accountability Contract
contract = FinOpsAccountabilityContract("EngineeringTeamA")
contract.set_cost_savings_goal(5.0)
metrics_data = {"date": "2023-10-15", "metric_value": 85}
contract.report_metrics(metrics_data)

# Output:
{
  "team_name": "EngineeringTeamA",
  "cost_savings_goal": 5.0,
  "cloud_utilization_metrics": [
    {"date": "2023-10-01", "metric_value": 76},
    {"date": "2023-10-08", "metric_value": 82},
    {"date": "2023-10-15", "metric_value": 85}
  ]
}
```

**Data:**

```json
{
  "team_name": "EngineeringTeamA",
  "cost_savings_goal": 5.0,
  "cloud_utilization_metrics": [
    {"date": "2023-10-01", "metric_value": 76},
    {"date": "2023-10-08", "metric_value": 82}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. > 500
6. < 500
7. Central Platform Team
8. Does it scale?
9. Results in high centralization
10. Requires more communicationand coordination efforts
11. Embedded Champions
12. Is local ownership critical?
13. Provides localizedaccountability, requires cleargoals
14. Central oversight with somelocal champions
15. Federated Ownership
16. How many teams are there?
17. Works well with distributedcontrol and shared metrics
18. Central coordination may beneeded for efficiency

