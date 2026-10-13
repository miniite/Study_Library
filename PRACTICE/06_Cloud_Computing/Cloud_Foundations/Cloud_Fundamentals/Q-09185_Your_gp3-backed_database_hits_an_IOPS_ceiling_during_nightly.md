---
type: "practice"
difficulty: "Deep"
---

# Your gp3-backed database hits an IOPS ceiling during nightly reports while CPU is idle. How would you tell it is IOPS and decide between raising provisioned IOPS or moving the report load?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Tail Latency Investigation](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Tail_Latency_Investigation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To diagnose whether your gp3-backed database is hitting an IOPS ceiling during nightly reports while CPU remains idle, we need to consider several factors including the nature of the workload, resource utilization patterns, and storage characteristics. The key steps involve monitoring, analysis, and strategic decision-making.

Firstly, let's understand what the question is really asking: we are trying to identify whether the database performance bottleneck is due to IOPS limitations or perhaps some other issue. We also need to decide between raising provisioned IOPS or moving the report load to optimize costs and performance.

#### Assumptions

The nightly reports are periodic tasks that run during times of low CPU usage.
We have access to monitoring tools like CloudWatch or Prometheus to gather necessary metrics.

Step-by-step from first principles, we would start by collecting detailed metrics on both IOPS and other relevant performance indicators:

Gather Metrics: Collect IOPS, read/write throughput, latency, CPU utilization, and memory usage during the report generation period.
Analyze Patterns: Determine if there are consistent patterns that suggest IOPS as a bottleneck (e.g., increased latency with high IOPS usage).
Isolate Workload Impact: Since CPU is idle, we can rule out CPU-related bottlenecks; focus on storage performance.
Compare to Baseline: Compare the metrics from the report generation period against baseline data when no reports are running.

After analysis, if we determine that IOPS are indeed the bottleneck, we need to decide between raising provisioned IOPS or moving the report load:

Raising Provisioned IOPS: This involves increasing the number of IOPS allowed by your gp3 instance. It’s a straightforward solution but can be expensive and may not always address the root cause.
Moving Report Load: Consider running reports during off-peak hours when CPU usage is lower, or use batch processing techniques to reduce resource contention.

Given our detailed analysis, let's assume we've confirmed that IOPS are the issue. We would then consider a strategy involving both raising provisioned IOPS and optimizing report execution times:

Increase IOPS Provisioning: Raise the IOPS limit for the database instance based on the required workload.
Optimize Report Execution:
Use batch processing to aggregate data before running reports, reducing the number of I/O operations.
Schedule reports during periods when CPU is idle but storage resources are underutilized.

#### Closing Summary
By collecting and analyzing detailed performance metrics, we can determine if IOPS are indeed the bottleneck. If so, raising provisioned IOPS may be necessary, but optimizing report execution times can also provide a more cost-effective solution.

#### Common Mistakes/Follow-Ups

Ignoring other potential bottlenecks like CPU or memory.
Failing to consider the long-term costs and scalability implications of increasing IOPS.
Not fully understanding the database's I/O patterns before making decisions.

**Comparison:**

| Time (UTC) | WriteOps |
|------------|----------|
| 2023-10-01T00:00Z | 100 |
| 2023-10-01T01:00Z | 150 |
| 2023-10-01T02:00Z | 200 |
| ...         | ...     |
| 2023-10-01T23:00Z | 800 |
| 2023-10-01T23:59Z | 900 |

**Code:**

```python
# Example Python script for collecting metrics using CloudWatch
import boto3

def collect_metrics(instance_id):
    cloudwatch = boto3.client('cloudwatch')

    response = cloudwatch.get_metric_data(
        MetricDataQueries=[
            {
                'Id': 'm1',
                'MetricStat': {
                    'Metric': {
                        'Namespace': 'AWS/RDS',
                        'MetricName': 'WriteOps',
                        'Dimensions': [
                            {'Name': 'DBInstanceIdentifier', 'Value': instance_id},
                        ]
                    },
                    'Period': 300,
                    'stat': 'Average'
                }
            },
        ],
        StartTime='2023-10-01T00:00:00Z',
        EndTime='2023-10-01T23:59:59Z'
    )

    return response['MetricDataResults'][0]['Values']

metrics = collect_metrics('my-database-instance')
print(metrics)
```

**Example input / output:**

```text
Input:
- Instance ID: my-database-instance

Output:
- Metric Values: [100, 150, 200, 300, 400, 500, 600, 700, 800, 900]
```

**Data:**

```json
{
  "metrics": [
    100,
    150,
    200,
    300,
    400,
    500,
    600,
    700,
    800,
    900
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is IOPS the bottleneck?
5. Raise provisioned IOPS
6. Schedule reports during idleperiods
7. Monitor performance andadjust as needed
8. End

