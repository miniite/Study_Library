---
type: "practice"
difficulty: "Intermediate"
---

# A stakeholder proposes a scenario with a response measure of 'zero failed deploys, ever.' How do you convert that unrealistic absolute into a measurable change-failure-rate and rollback-time target the pipeline architecture can actually be evaluated against?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-2/3 · [↑ Sensitivity Points](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Sensitivity_Points.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When stakeholders propose an unrealistic requirement like "zero failed deploys, ever," we need to translate this aspiration into quantifiable metrics that a DevOps pipeline can realistically aim for. This involves understanding the key performance indicators (KPIs) and trade-offs involved in deployment strategies. Let's break down the process step by step.

Firstly, the request of zero failed deploys is essentially asking us to ensure every single deployment is successful without any rollbacks. However, achieving this can be extremely challenging due to various factors like unpredictable real-world usage or unforeseen bugs. Instead, we should focus on defining a more realistic target that measures the frequency and impact of failures.

To do this, we need to evaluate what "failure" means in the context of our application—whether it's a deployment failure, rollback time, or downtime during deployments. We'll use the concept of change-failure-rate (CFR) and rollback-time as primary metrics.

Let's define these terms:

Change-Failure-Rate (CFR): The number of failed deployments per total number of deployments.
Rollback-Time: Time taken to revert to a previous version in case of failure.

Next, we need to understand the current state of our deployment pipeline. For instance, if currently 10% of deployments fail and it takes an average of 30 minutes to rollback, this gives us a baseline from which to improve.

Now, let's lay out a step-by-step approach:

Assess Current Performance: Gather data on current deployment failures and their impact.
Define Improvement Goals: Set realistic goals based on industry standards or best practices.
For example, an acceptable CFR might be less than 5% over the next quarter with an average rollback time under 60 minutes.
Implement Strategies:
Introduce pre-deployment tests and automated checks.
Implement blue-green deployments to minimize downtime.
Use canary releases to gradually roll out changes.

By focusing on these metrics, we can create a more achievable set of goals that align with the initial requirement but are grounded in practical realities. This approach ensures that our pipeline architecture is evaluated against realistic targets.

In summary, converting "zero failed deploys" into measurable targets involves defining key performance indicators like change-failure-rate and rollback-time, assessing current performance, setting realistic improvement goals, and implementing strategies to achieve those goals.

**Comparison:**

```
Metric	Current Value	Target Value
Change-Failure-Rate (CFR)	5%	<5%
Average Rollback Time (minutes)	30	<60
```

**Code:**

```python
# Example code for calculating Change-Failure-Rate (CFR)
def calculate_cfr(failed_deployments, total_deployments):
    return failed_deployments / total_deployments

# Example data
failed_deployments = 5
total_deployments = 100
cfr = calculate_cfr(failed_deployments, total_deployments)
print(f"Current CFR: {cfr}")
```

**Example input / output:**

```text
**Input Data:**
- Total Deployments: 100
- Failed Deployments: 5
- Average Rollback Time: 30 minutes

**Output Metrics:**
- Current Change-Failure-Rate (CFR): 5%
- Current Average Rollback Time: 30 minutes

**Target Goals:**
- Target CFR: Less than 5% over the next quarter
- Target Average Rollback Time: Under 60 minutes
```

**Data:**

```json
{
  "current_deployments": {
    "total_deployments": 100,
    "failed_deployments": 5,
    "average_rollback_time_minutes": 30
  }
}
```

**Flow:**

1. Yes
2. No
3. Current State
4. Is Current CFR and RBTAchievable?
5. Set Realistic Goals
6. Improve DeploymentProcesses
7. Metric Tracking & ContinuousImprovement
8. Implement New Strategies

