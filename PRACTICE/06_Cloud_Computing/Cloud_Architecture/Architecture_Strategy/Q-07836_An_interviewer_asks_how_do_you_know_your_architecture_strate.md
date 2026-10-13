---
type: "practice"
difficulty: "Deep"
---

# An interviewer asks: how do you know your architecture strategy is actually working? What leading and lagging indicators do you track at the org level?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Balancing Architecture Pillars](<../../../../06_Cloud_Computing/Cloud_Architecture/Architecture_Strategy/Balancing_Architecture_Pillars.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When evaluating whether a cloud architecture strategy is effective, it's crucial to establish both leading and lagging indicators that reflect different aspects of your organization’s performance. Leading indicators focus on proactive measures that signal future success, while lagging indicators measure historical outcomes.

At the org level, I track a comprehensive set of metrics across several domains: technical, operational, cost, and business impact. Here's how I break it down:

Technical Indicators:

Latency Metrics: Ensure that user experience is optimal by monitoring response times for critical services.
Resource Utilization: Keep an eye on CPU, memory, and storage usage to ensure efficient resource allocation.

Operational Indicators:

Incident Frequency and Resolution Time: Shorter resolution times indicate a more resilient infrastructure.
Change Management Effort: Lower effort in change management suggests improved processes and automation.

Cost Indicators:

Budget Compliance: Ensure spending aligns with predefined budgets to avoid financial overruns.
Cost Per Unit of Work: Measure the cost efficiency by comparing expenditure against work output or service level agreements (SLAs).

Business Impact Indicators:

Customer Satisfaction and Retention Rates: Higher satisfaction and retention imply better service quality, which can be directly linked to your architecture strategy.
Revenue Growth and Cost Savings: Track how the infrastructure impacts revenue generation and cost reductions.

To implement this strategy, I would set up a dashboard that aggregates these metrics from various tools such as CloudWatch, Stackdriver, and custom monitoring solutions. The key is to ensure the data is actionable, meaning it should trigger alerts or notifications when thresholds are breached, allowing for timely intervention.

**Comparison:**

```
Metric	Description	Leading/Lagging
Response Time	Average time taken to serve requests	Lagging
Resource Utilization	Proportion of resources used	Leading
```

**Code:**

```python
# Example of setting up an alert based on CPU usage threshold in Python
import boto3

def check_cpu_usage(bucket_name='logs-bucket', alarm_threshold=80):
    cloudwatch = boto3.client('cloudwatch')

    response = cloudwatch.get_metric_statistics(
        Namespace='AWS/EC2',
        MetricName='CPUUtilization',
        Dimensions=[{'Name': 'InstanceId', 'Value': 'your-instance-id'}],
        StartTime='2023-10-01T00:00:00Z',
        EndTime='2023-10-07T00:00:00Z',
        Period=86400,
        Statistics=['Average'],
    )

    for point in response['Datapoints']:
        if point['Average'] > alarm_threshold:
            print(f"CPU usage is high on instance {point['Dimensions'][0]['Value']}")

check_cpu_usage()
```

**Example input / output:**

```text
Input:
- CPU Utilization data over the past week for an instance.

Output:
- Alert if CPU usage exceeds 80%.
```

**Data:**

```json
{
  "metrics": [
    {
      "name": "ResponseTime",
      "unit": "ms",
      "thresholds": {"high": 250, "low": 100}
    },
    {
      "name": "ResourceUtilization",
      "unit": "%",
      "thresholds": {"high": 90, "low": 30}
    }
  ]
}
```

**Flow:**

1. Set Up Metrics
2. Define Technical Indicators
3. CPU, Memory, Storage Usage
4. Define Operational Indicators
5. Incident Frequency, ResolutionTime, Change ManagementEffort
6. Define Cost Indicators
7. Budget Compliance, Cost PerUnit of Work
8. Define Business ImpactIndicators
9. Customer Satisfaction,Retention Rates, RevenueGrowth, Cost Savings

