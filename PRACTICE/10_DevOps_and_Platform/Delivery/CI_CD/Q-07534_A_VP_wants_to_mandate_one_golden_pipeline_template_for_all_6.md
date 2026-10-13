---
type: "practice"
difficulty: "Expert"
---

# A VP wants to mandate one golden pipeline template for all 60 teams; you know mandates get routed around. How do you design the paved road so 90% adopt it voluntarily, fund the platform to keep it best-in-class, and measure adoption honestly rather than by decree compliance?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Golden Pipelines and Templates](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/Golden_Pipelines_and_Templates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need a multi-faceted approach that balances mandates with voluntary adoption, ensures continuous improvement through robust funding mechanisms, and measures adoption in a way that provides actionable insights rather than mere compliance.

Firstly, the mandate to use one golden pipeline template is driven by the desire for standardization across 60 teams. This template must be designed with the utmost care to ensure it covers all necessary stages of CI/CD, from initial setup to post-deployment monitoring, and should be flexible enough to accommodate varying team needs while maintaining consistency.

We will design a "paved road" by:

Educating Teams: Providing comprehensive training sessions and resources to help teams understand the benefits of standardization.
Building Incentives: Creating financial incentives for adopting and optimizing the pipeline template, such as cost savings from reduced overhead and improved efficiency.
Fostering Collaboration: Encouraging knowledge sharing through regular meetups and a shared repository where best practices can be continuously updated.
Setting Clear Metrics: Defining measurable success criteria that go beyond mere compliance, such as reduced deployment time or increased software quality.

To ensure the platform remains best-in-class, we will establish dedicated funding mechanisms:

Allocate a portion of the overall DevOps budget specifically for pipeline improvements and innovation.
Leverage external tools and services to enhance the template's capabilities while maintaining cost-effectiveness.

Finally, measuring adoption honestly involves setting up a system that tracks real-world usage and impact:

Implement metrics like DORA (DevOps Research and Assessment) metrics to gauge deployment frequency, lead time, change fail percentage, etc.
Conduct periodic reviews of teams' pipeline configurations to ensure they are aligned with the template.
Use qualitative feedback from teams on how the template has impacted their workflows.

**Comparison:**

```
Team	Deployment Frequency (times/month)	Lead Time (min)
DevTeam1	5	30
DevTeam2	4	45
```

**Code:**

```python
# Example: Metrics Collection Script for Pipeline Template Usage
def collect_metrics(pipeline_config):
    metrics = {
        'deployment_frequency': 0,
        'lead_time': 0,
        # ... other DORA metrics
    }
    # Logic to update metrics based on pipeline configuration and execution data
    return metrics

def main():
    pipelines = fetch_pipelines()  # API to fetch all pipeline configurations
    for pipeline in pipelines:
        metrics = collect_metrics(pipeline)
        log_metrics(metrics)  # Log metrics to database or analytics tool

main()
```

**Example input / output:**

```text
Input:
- Fetch all pipeline configurations from the CI/CD system

Output:
{
  "metrics": {
    "DevTeam1": {"deployment_frequency": 5, "lead_time": 30},
    "DevTeam2": {"deployment_frequency": 4, "lead_time": 45}
  }
}

Request: Log these metrics to a central database for analysis.
Response: Metrics successfully logged with timestamps and associated team identifiers.
```

**Data:**

```json
{
  "pipeline_configs": [
    {
      "team_name": "DevTeam1",
      "deployment_frequency": 5,
      "lead_time": 30,
      "change_fail_percentage": 2.0
    },
    {
      "team_name": "DevTeam2",
      "deployment_frequency": 4,
      "lead_time": 45,
      "change_fail_percentage": 1.5
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Fetch Pipeline Configurations
4. Are Metrics Available?
5. Log Metrics to Database
6. Send Alerts to Teams forConfiguration Updates
7. Analyze Metrics and ProvideFeedback
8. Fund Platform ImprovementsBased on Feedback
9. Celebrate Successes and SetNew Goals

