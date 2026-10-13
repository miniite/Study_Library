---
type: "practice"
difficulty: "Expert"
---

# A bar-raiser asks: you've made GPU-hours a chargeable resource, and now research teams game it by requesting fewer GPUs but running longer, or by hoarding a reserved allocation. How do you design a GPU-cost-visibility and quota system that's hard to game by construction rather than by policing?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ GPU Capacity Economics](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/GPU_Capacity_Economics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In this scenario, we need to design a robust GPU cost visibility and quota system that prevents gaming the system through over-provisioning or under-utilization of resources. The key is to create mechanisms that align researchers’ incentives with efficient resource utilization while maintaining transparency.

#### Triage

The question revolves around designing an effective FinOps (Financial Operations) system for managing cloud-based GPU resources. Specifically, we need to address gaming behaviors by creating a system that inherently discourages inefficient use of resources without the need for extensive policing or enforcement.

#### Assumptions
and First Principles
We are working with a dynamic allocation model where researchers can request GPUs.
The goal is to minimize wasted compute time while maximizing overall productivity.
Researchers have varying workloads, some may require more GPU time than others.
The system should provide clear visibility into resource utilization and costs.
Step-by-Step Solution
1. Define Quotas Based on Historical Data

First, we need a way to set initial quotas for each research team based on historical data. For example:

SQL
SELECT
    team_id,
    SUM(gpu_hours) AS total_gpu_hours
FROM
    gpu_usage_history
GROUP BY
    team_id;

This query can help us understand the typical GPU usage patterns of different teams.

2. Implement Elastic Scaling Policies

Use dynamic scaling policies to automatically adjust the number of GPUs based on current workload:

PYTHON
def scale_gpus(team_id, min_gpu_count, max_gpu_count):
    # Query recent GPU usage data for the team
    recent_usage = get_recent_gpu_usage_data(team_id)

    if recent_usage < 50:  # Example threshold
        scale_down(min_gpu_count)
    elif recent_usage > 75:
        scale_up(max_gpu_count)
3. Introduce Performance Metrics and Incentives

Develop performance metrics that reward teams for efficient use of resources. For instance, we can introduce a metric called "GPU Efficiency Score":

PYTHON
def calculate_efficiency_score(team_id):
    total_gpu_hours = get_total_gpu_hours(team_id)
    total_computing_time = get_total_computing_time(team_id)
    efficiency_score = total_gpu_hours / total_computing_time
    return efficiency_score

Teams with higher efficiency scores can receive benefits such as additional budget or priority access to resources.

4. Implement Cost-Based Quotas

Set cost-based quotas where the maximum allowable GPU-hours are tied directly to the research team's allocated budget:

PYTHON
def set_cost_based_quotas(team_id, budget):
    # Calculate hourly rate based on total monthly spend
    total_monthly_spend = get_total_monthly_spend()
    hourly_rate = total_monthly_spend / (30 * 24)

    max_gpu_hours = budget / hourly_rate
    return max_gpu_hours
5. Continuous Monitoring and Feedback Loops

Implement continuous monitoring to track real-time GPU usage and provide feedback to teams:

PYTHON
def monitor_and_feedback(team_id):
    current_usage = get_current_gpu_usage(team_id)

    if current_usage > set_cost_based_quotas(team_id, team_budget):
        send_email_alert(f"Team {team_id} has exceeded their budget.")
6. Transparency and Reporting

Provide regular reports to all teams on their GPU usage and costs:

PYTHON
def generate_reports():
    for team in get_all_teams():
        report = f"Team {team.id}: \n- Total GPU Hours: {team.total_gpu_hours}\n- Cost: ${team.cost}"
        send_report(report)

#### Closing Summary

By implementing a combination of dynamic scaling, cost-based quotas, performance metrics, and continuous monitoring, we can design a system that discourages gaming the system. This approach ensures that teams are incentivized to use resources efficiently while maintaining transparency and visibility.

#### Common Mistakes
and Follow-Ups
Static Quotas: Failing to dynamically adjust quotas based on real-time usage can lead to underutilization or waste.
Lack of Feedback Mechanisms: Without providing timely feedback, teams may continue inefficient practices.
Insufficient Historical Data: Relying solely on initial data without regular updates and adjustments can result in inaccurate quota settings.

To further refine this solution, we could consider integrating machine learning models to predict future usage patterns or implementing more granular cost controls based on specific job types.

**Comparison:**

```
Team ID	Total GPU Hours (History)	Cost-Based Quota
1	200	277.8
2	500	694.4
Team ID	Current Usage	Efficiency Score
1	40	5
2	80	6.25
```

**Code:**

```python
def scale_gpus(team_id, min_gpu_count, max_gpu_count):
    recent_usage = get_recent_gpu_usage_data(team_id)
    if recent_usage < 50:
        scale_down(min_gpu_count)
    elif recent_usage > 75:
        scale_up(max_gpu_count)

def calculate_efficiency_score(team_id):
    total_gpu_hours = get_total_gpu_hours(team_id)
    total_computing_time = get_total_computing_time(team_id)
    efficiency_score = total_gpu_hours / total_computing_time
    return efficiency_score

def set_cost_based_quotas(team_id, budget):
    total_monthly_spend = get_total_monthly_spend()
    hourly_rate = total_monthly_spend / (30 * 24)
    max_gpu_hours = budget / hourly_rate
    return max_gpu_hours

def monitor_and_feedback(team_id):
    current_usage = get_current_gpu_usage(team_id)
    if current_usage > set_cost_based_quotas(team_id, team_budget):
        send_email_alert(f"Team {team_id} has exceeded their budget.")

def generate_reports():
    for team in get_all_teams():
        report = f"Team {team.id}: \n- Total GPU Hours: {team.total_gpu_hours}\n- Cost: ${team.cost}"
        send_report(report)
```

**Example input / output:**

```text
# Example input and output

# Initial setup for team 1
scale_gpus(1, 4, 8)
set_cost_based_quotas(1, 1000)

# Feedback loop after monitoring usage
monitor_and_feedback(1) # Email sent if budget exceeded

# Generate reports for all teams
generate_reports()
```

**Data:**

```json
{
  "gpu_usage_history": [
    {"team_id": 1, "gpu_hours": 200},
    {"team_id": 2, "gpu_hours": 500}
  ],
  "recent_gpu_usage_data": [
    {"team_id": 1, "gpu_hours": 40},
    {"team_id": 2, "gpu_hours": 80}
  ],
  "total_monthly_spend": 3600,
  "team_budgets": {
    "team_1": 1000,
    "team_2": 2500
  }
}
```

**Flow:**

1. Yes
2. No
3. Team Request
4. Is Usage Exceeding Quota?
5. Send Alert
6. Monitor Usage & Feedback
7. Generate Reports

