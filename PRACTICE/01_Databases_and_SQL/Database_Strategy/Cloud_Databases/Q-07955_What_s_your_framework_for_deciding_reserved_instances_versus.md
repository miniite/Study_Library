---
type: "practice"
difficulty: "Deep"
---

# What's your framework for deciding reserved instances versus savings plans versus on-demand across a fluctuating RDS/Aurora fleet, and how do you avoid over-committing the org?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cloud Database Architecture](<../../../../01_Databases_and_SQL/Database_Strategy/Cloud_Databases/Cloud_Database_Architecture.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When architecting a cloud database strategy that involves RDS/Aurora in a fluctuating fleet, the primary goal is to balance cost efficiency with operational reliability. The framework for deciding between reserved instances (RIs), savings plans (SPs), and on-demand requires a thorough understanding of usage patterns, cost implications, and service level agreements (SLAs). Here’s how you would approach this:

#### Overview
The key idea is to leverage a hybrid strategy that maximizes cost savings while ensuring high availability and compliance with SLAs.

#### Triage
The question is probing your ability to design a resilient and cost-effective database infrastructure, considering both short-term fluctuations in demand and long-term usage trends.
Step-by-Step Solution from First Principles

#### Assumptions

The fleet size can vary significantly over time due to business needs.
There are clear SLAs for uptime and performance that must be met.
Cost is a significant factor, but not the only one; reliability and service quality also matter.

Data Collection:

Collect historical usage data: total number of instances, duration of use per instance, and specific periods of high or low demand.
Identify patterns in usage to predict future needs accurately.

Cost Analysis:

Calculate the cost for each type of pricing model (RIs, SPs, on-demand).
Use AWS Cost Explorer or similar tools to analyze historical costs and forecast future expenses based on different scenarios.

SLA Compliance:

Ensure that SLAs are met for critical applications.
Define thresholds for acceptable downtime; ensure RIs and SPs meet these requirements.

Hybrid Strategy:

RDS/Aurora Reserved Instances (RI): Allocate RIs to mission-critical, predictable workloads where the cost savings justify the upfront investment. For example, a data warehouse or analytics workload that runs 24/7.
Aurora Savings Plans (SP): Optimize for longer-term use cases with consistent demand patterns. SPs offer significant discounts and are ideal for regular database instances used in development or testing environments.
On-Demand Instances: Use on-demand when there is no predictable usage pattern, ensuring flexibility to scale up or down as needed without the commitment of RIs or SPs.

Dynamic Management:

Implement automation scripts using AWS Lambda and CloudFormation to dynamically adjust instance sizes based on real-time demand.
Leverage Auto Scaling policies to ensure resources are optimized during peak times while maintaining cost efficiency.

Cost Optimization Tools:

Use Amazon EC2 Cost Explorer, RDS Advisor, and other AWS services to monitor and optimize costs continuously.
Implement spot instances where the workload can tolerate occasional interruptions for significant cost savings.

Regular Review and Adjustment:

Conduct periodic reviews of the usage patterns and make adjustments as needed based on new data or business changes.
Ensure that SLA compliance is maintained, especially during major organizational shifts.

#### Closing Summary

By adopting a hybrid strategy that leverages RIs for predictable workloads, SPs for consistent demand, and on-demand instances for flexible scaling needs, you can achieve both cost efficiency and reliability. Regular monitoring and adjustment are crucial to ensure optimal resource utilization and SLA compliance.

#### Common Mistakes/Follow-Ups
Over-committing resources can lead to unnecessary costs if not managed properly.
Underestimating the impact of unpredictable workloads on SLAs.
Neglecting the potential for spot instances in non-critical applications, which can significantly reduce costs while maintaining acceptable performance levels.

**Comparison:**

```
SLA Category	Metric	Threshold	RI/SP Requirement
Uptime	Downtime	< 1% per month	High availability
Performance	Latency	< 20ms	Standard SLA
Compliance	Audit Passes	100%	Regular testing
```

**Code:**

```python
# Example Python script for dynamic instance scaling using Boto3 (AWS SDK)
import boto3

def scale_instances(instance_type, count):
    ec2 = boto3.resource('ec2')
    instances = ec2.create_instances(
        ImageId='ami-0c55b159cbfafe1f0',  # Example AMI ID
        MinCount=count,
        MaxCount=count,
        InstanceType=instance_type
    )
    return [instance.id for instance in instances]

# Example usage
scale_instances('t3.micro', 2)
```

**Example input / output:**

| Scenario | On-Demand Cost Estimate | RI Commitment Cost | SP Savings Potential |
|----------|------------------------|--------------------|---------------------|
| Data Warehouse | $240,000/year | $150,000/year (3-year commitment) | $87,000/year savings |
| Application DB | $60,000/year | $20,000/year (yearly commitment) | $40,000/year savings |

**Data:**

```json
{
  "ri_usage_patterns": {
    "data_warehouse": {
      "instance_types": ["r5.4xlarge"],
      "duration_days": 365,
      "commitment_cost": 10000
    },
    "application_db": {
      "instance_types": ["db.r5.large"],
      "duration_days": [90, 180, 270],
      "commitment_cost": 1000
    }
  },
  "sp_usage_patterns": {
    "test_environment": {
      "instance_types": ["db.t3.small"],
      "usage_hours_per_day": 16,
      "duration_months": [3, 6, 9]
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Monitor Usage Patterns
6. Is Predictable?
7. Allocate RIs
8. Is Consistent Demand?
9. Allocate SPs
10. Use On-Demand Instances

