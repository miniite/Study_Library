---
type: "practice"
difficulty: "Expert"
---

# Your architecture uses a lot of managed services for velocity, but the aggregate managed-service premium is now questioned by finance. How do you build the honest total-cost-of-ownership comparison (including engineering time) of managed versus self-hosted?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ TCO of Self-managed Platforms](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/TCO_of_Self-managed_Platforms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the concern about the managed-service premium in your architecture, we need to conduct a comprehensive Total Cost of Ownership (TCO) analysis that includes both direct and indirect costs such as engineering time, maintenance, support, and potential future expenses. Here’s how you can approach this problem step-by-step.

Firstly, let's define what TCO entails:

Direct Costs: These include the subscription fees for managed services.
Indirect Costs: These encompass engineering effort required to manage, monitor, and maintain these services, as well as potential costs associated with any future issues or downtime.

Now, we need to break down the steps involved in performing this analysis:

Identify Direct Costs:

Gather subscription details for each managed service used.
Document any additional charges like API requests, storage usage, etc.

Engineering Effort Analysis:

Calculate the time spent on provisioning, configuring, and maintaining these services.
Estimate the effort required to handle issues such as performance tuning, security audits, and updates.
Include the cost of any dedicated teams or individuals managing these services.

Future Costs:

Assess potential future costs due to scaling needs, service disruptions, or maintenance.

#### Cons
ider the impact on overall infrastructure stability and risk mitigation strategies.

Self-Hosted vs Managed Comparison:

Evaluate the TCO for a self-hosted solution that would replicate what is currently managed.
Compare this with the current managed service costs to determine savings or additional expenses.

By following these steps, we can provide an honest and detailed comparison that addresses the finance team’s concerns. This approach not only quantifies direct and indirect costs but also provides insights into long-term benefits and risks associated with both approaches.

**Comparison:**

| Task                       | Managed Service Effort (hours) | Self-Hosted Effort (hours) |
|----------------------------|--------------------------------|---------------------------|
| Provisioning               | 10                             | -                         |
| Configuration              | 20                             | -                         |
| Security Audits            | -                              | 50                        |
| Setup Environment          | -                              | 30                        |

**Code:**

```python
# Example function to calculate engineering effort in hours
def estimate_engineering_effort(tasks, avg_hourly_rate):
    total_hours = sum(task['hours'] for task in tasks)
    return total_hours * avg_hourly_rate

# Sample data
tasks_managed_service = [
    {'task': 'Provisioning', 'hours': 10},
    {'task': 'Configuration', 'hours': 20}
]

tasks_self_hosted = [
    {'task': 'Setup Environment', 'hours': 30},
    {'task': 'Security Audits', 'hours': 50}
]

avg_hourly_rate = 100  # $ per hour

managed_service_effort = estimate_engineering_effort(tasks_managed_service, avg_hourly_rate)
self_hosted_effort = estimate_engineering_effort(tasks_self_hosted, avg_hourly_rate)

print(f"Estimated engineering effort for managed service: {managed_service_effort} hours")
print(f"Estimated engineering effort for self-hosted solution: {self_hosted_effort} hours")
```

**Example input / output:**

```text
Input:
- managed_service_effort(tasksManagedService, avg_hourly_rate)
- self_hosted_effort(tasksSelfHosted, avg_hourly_rate)

Output:
Estimated engineering effort for managed service: 3000 hours (assuming $100/hour)
Estimated engineering effort for self-hosted solution: 8000 hours (assuming $100/hour)
```

**Data:**

```json
{
  "tasksManagedService": [
    {"task": "Provisioning", "hours": 10},
    {"task": "Configuration", "hours": 20}
  ],
  "tasksSelfHosted": [
    {"task": "Setup Environment", "hours": 30},
    {"task": "Security Audits", "hours": 50}
  ]
}
```

**Flow:**

1. Define Direct Costs
2. Calculate Engineering Effort
3. Estimate Future Costs
4. Evaluate Self-Hosted vsManaged
5. Provide Honest TCOComparison

