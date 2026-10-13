---
type: "practice"
difficulty: "Deep"
---

# Your warehouse charges by compute-second and a handful of runaway analyst queries dominate spend, but blocking them kills legitimate exploration. Design the query-governance system — resource monitors, per-user credit quotas, query timeouts, workload isolation — that curbs runaway cost without gatekeeping analysis.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Query Cost Guardrails](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Query_Cost_Guardrails.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of runaway analyst queries while ensuring cost control and supporting legitimate analysis, we need to design a comprehensive FinOps & Cloud Cost Governance system. This involves setting up resource monitors, implementing per-user credit quotas, enforcing query timeouts, and managing workload isolation. The goal is to balance cost efficiency with analytical flexibility.

The core idea is to use a layered approach:

Resource Monitors: Continuously track usage metrics.
Per-User Credit Quotas: Limit the amount of compute seconds each user can consume.
Query Timeouts: Automatically terminate long-running or resource-intensive queries.
Workload Isolation: Ensure that one query does not impact others.

Let's break down each component:

#### Triaging the Question

The interviewer is probing our understanding of:

The technical aspects of AWS (or a similar cloud provider) and how to manage costs effectively.
Strategies for balancing cost control with analytical flexibility.
Best practices in designing a scalable, maintainable, and user-friendly governance system.

#### Assumptions
We are working on an AWS environment, using services like Amazon Redshift or Athena.
The system should be able to handle multiple users concurrently.
Users will have varying levels of access and resource requirements.
There is a need for real-time monitoring and alerting mechanisms.

#### First-Principles Solution

Resource Monitors

Implement CloudWatch metrics to track compute seconds, query duration, and other relevant KPIs.
Set up alarms to notify the operations team of unusual spikes in usage.

Per-User Credit Quotas

Define a credit system where each user has a monthly or daily limit on compute seconds.
Integrate this quota with IAM roles and policies for enforcement.
Provide users with visibility into their remaining credits through a dashboard.

Query Timeouts

Establish default timeouts based on historical data of acceptable query durations.
Allow administrators to adjust these timeouts as needed without manual intervention from users.

Workload Isolation

Use virtual clusters or dedicated nodes in Redshift to isolate workloads.
Implement network segmentation to prevent resource contention between different queries.

#### Closing Summary
and Common Mistakes

This system ensures that runaway queries do not dominate costs while supporting legitimate analysis by providing clear, enforceable rules. However, common mistakes include:

Not setting up adequate monitoring and alerting mechanisms.
Overly aggressive credit quotas that could stifle user productivity.
Failing to provide users with enough context about their usage and remaining credits.
Common Follow-Ups
How do you handle the transition from a per-user quota system to a more dynamic, self-service approach?
What are some best practices for tuning query timeouts without affecting legitimate analysis?
How would you integrate this system into an existing data pipeline?

**Comparison:**

```
Metric	Description	Threshold (Seconds)
TotalComputeSec	Sum of all compute seconds used	250,000
QueryDuration	Average query duration	120
```

**Code:**

```python
# Example Python code snippet for setting up CloudWatch alarms in AWS using Boto3
import boto3

def create_cloudwatch_alarm(client, metric_name, threshold, comparison_operator='GreaterThanThreshold'):
    alarm = client.put_metric_alarm(
        AlarmName=f"HighComputeSeconds-{metric_name}",
        MetricName=metric_name,
        Namespace="AWS/Redshift",
        Statistic="Sum",
        Period=300,
        EvaluationPeriods=1,
        Threshold=threshold,
        ComparisonOperator=comparison_operator,
        ActionsEnabled=True
    )
```

**Example input / output:**

| User | Quota Type | Limit (Compute Seconds) |
|------|-----------|------------------------|
| Alice | Monthly   | 50,000                 |
| Bob   | Daily     | 10,000                 |

**Data:**

```json
{
  "userQuotas": [
    { "username": "Alice", "monthlyLimit": 50000 },
    { "username": "Bob", "dailyLimit": 10000 }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Query Initiated
6. User has Credits?
7. Execute Query
8. Alert Admin
9. Monitor Query Duration andMemory Usage
10. Timeout Exceeded?
11. Kill Query, Notify User

