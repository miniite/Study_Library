---
type: "practice"
difficulty: "Intermediate"
---

# Describe how you'd set up per-team showback in an org that isn't ready for chargeback. What report do teams get, how often, and how do you keep them from ignoring it?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Usage-based Cost Surprises](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Usage-based_Cost_Surprises.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When setting up per-team showback in an organization not yet ready for full chargeback, the goal is to provide visibility into cloud cost usage while minimizing overhead and resistance. The key steps involve creating a simplified report that shows each team's resource consumption, setting up regular intervals for reporting (e.g., monthly), and establishing clear communication channels.

The main idea here is to leverage existing tools and processes within the organization to gather and present data in a way that’s both transparent and actionable without requiring significant additional infrastructure or complexity. This approach helps teams understand their costs, which can drive more responsible resource management over time.

**Comparison:**

| Team | Month       | Cost     |
|------|------------|----------|
| DevOps   | October 2023 | $7,800.00 |
| Sales    | October 2023 | $10,500.00 |

**Code:**

```python
# Example of how cost data might be aggregated using AWS CloudWatch Logs
import boto3

def fetch_team_cost_data(team_name):
    client = boto3.client('ce')
    response = client.get_cost_and_usage(
        TimePeriod={
            'Start': '2023-10-01',
            'End': '2023-10-31'
        },
        Granularity='MONTHLY',
        Metrics=['UnblendedCost'],
        Filters={
            'Tags': {
                'Key': 'Team',
                'Values': [team_name]
            }
        }
    )
    return response['ResultsByTime'][0]['Total']['UnblendedCost']['Amount']
```

**Example input / output:**

```text
Input:
- Team name (e.g., 'Engineering')
- Date range for cost data retrieval

Output:
{
  "Team": "Engineering",
  "Month": "October 2023",
  "Cost": "$15,400.00"
}
```

**Data:**

```json
{
  "Report": [
    {
      "Team": "Engineering",
      "Month": "October 2023",
      "Cost": "$15,400.00"
    },
    {
      "Team": "Marketing",
      "Month": "October 2023",
      "Cost": "$8,700.00"
    }
  ]
}
```

**Flow:**

1. No
2. Yes
3. Data Fetch
4. Is Team Ready?
5. Generate Simplified Report
6. Implement Chargeback System
7. Send Monthly Reports toTeams
8. Monitor Compliance andUsage

