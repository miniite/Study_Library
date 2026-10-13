---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd govern the cost of feature-rich observability where the vendor charges per-host, per-GB, and per-custom-metric simultaneously. How do you decompose that bill and decide which pricing dimension to attack first?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Governing Data Platform Costs](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Governing_Data_Platform_Costs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To govern the cost of a feature-rich observability platform where vendors charge per-host, per-GB, and per-custom-metric simultaneously, we need to break down the bill into its components, understand the usage patterns, and prioritize optimization efforts based on both financial impact and operational efficiency. The key is to identify which dimension—host, storage, or custom metrics—is driving the most significant cost.

#### Triage
Understanding the Question

The interviewer is probing your ability to:

Decompose complex costs: Understand how different usage factors contribute to overall expenditure.
Analyze and prioritize optimizations: Determine which aspects of observability are most costly and need attention first.
Balance business needs with cost management: Ensure that the chosen optimization strategy aligns with both technical requirements and budget constraints.

#### Assumptions
The observability platform is hosted in a cloud environment, likely AWS or Azure.
We have access to detailed usage metrics and cost breakdown reports from the vendor.
There are no regulatory or compliance constraints that prevent certain optimizations.

#### First Principles Solution

Collect Detailed Usage Metrics:

Gather data on hosts, storage (GB), and custom metrics used for observability.
Use tools like AWS Cost Explorer, Azure Cost Management, or third-party billing dashboards to get a granular view of costs per service and resource.

Analyze Cost Drivers:

Calculate the total cost attributed to each dimension: hosts, storage, and custom metrics.
Identify which component is responsible for the largest share of the bill.

Prioritize Optimization Based on Impact:

Hosts: If multiple services or applications are running on a single host, consider containerization to optimize resource usage.
Storage (GB): Minimize unnecessary log retention periods and implement efficient storage solutions like S3 Glacier for archival data.
Custom Metrics: Optimize custom metric collection by reducing the frequency of metric gathering or using more efficient metric protocols.

Implement Cost Governance Policies:

Set up alerts and notifications to flag excessive usage in real-time.
Implement budget controls at both the project level and the individual service level.
Use automated cost optimization tools that can dynamically scale resources based on demand and historical patterns.

Monitor and Iterate:

Continuously monitor the effectiveness of optimizations through regular audits and updates to the cost governance strategy.
Adjust strategies as new usage patterns emerge or as vendor pricing models evolve.

#### Closing Summary

Deciding which dimension to attack first requires a holistic approach that combines detailed data analysis, understanding of business requirements, and strategic optimization. By breaking down the bill into its components, we can prioritize efforts effectively and ensure that cost governance aligns with both financial goals and operational needs.

#### Common Mistakes/Follow-Ups
Overlooking Data Retention: Failing to optimize storage by reducing unnecessary data retention periods.
Neglecting Host Utilization: Not considering containerization or microservices architectures for better host utilization.
Ignoring Custom Metric Efficiency: Overlooking the impact of custom metric collection frequency and protocol efficiency.

**Comparison:**

```
Dimension	Usage	Cost
Hosts	3 instances	$X
Storage	2500GB (archive)	$Y
Custom Metrics	800 metrics/sec	$Z
```

**Code:**

```python
# Example script to collect cost metrics using AWS Cost Explorer API
import boto3

client = boto3.client('ce')

response = client.get_cost_and_usage(
    TimePeriod={
        'Start': '2023-01-01',
        'End': '2023-01-31'
    },
    Granularity='MONTHLY',
    Metrics=['UnblendedCost']
)

print(response['ResultsByTime'][0]['Total']['UnblendedCost']['Amount'])
```

**Example input / output:**

```text
Input:
- Hosts: i-0123456789abcdef0 (2), i-0234567890abcdef1 (1)
- Storage: logs-bucket (500GB, 365 days retention), data-bucket (2000GB, 90 days retention)
- Custom Metrics: user_activity (every 10s, count 500), request_latency (every 5s, count 300)

Output:
- Total Cost: $X
- Cost Breakdown:
  - Hosts: $Y
  - Storage: $Z
  - Custom Metrics: $A

Decisions:
- Optimize hosts by using containers.
- Archive logs in cheaper storage tiers.
- Reduce the frequency of custom metric collection to save costs.
```

**Data:**

```json
{
  "host_usage": [
    {"instance_id": "i-0123456789abcdef0", "metrics": ["cpu", "memory"], "count": 2},
    {"instance_id": "i-0234567890abcdef1", "metrics": ["network"], "count": 1}
  ],
  "storage_usage": [
    {"bucket_name": "logs-bucket", "size_gb": 500, "retention_period_days": 365},
    {"bucket_name": "data-bucket", "size_gb": 2000, "retention_period_days": 90}
  ],
  "custom_metric_usage": [
    {"metric_name": "user_activity", "collection_frequency_seconds": 10, "count": 500},
    {"metric_name": "request_latency", "collection_frequency_seconds": 5, "count": 300}
  ]
}
```

**Flow:**

1. Collect Detailed Usage Metrics
2. Analyze Cost Drivers
3. Identify Cost Drivers
4. Prioritize Optimization Basedon Impact
5. Implement Cost GovernancePolicies
6. Monitor and Iterate

