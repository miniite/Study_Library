---
type: "practice"
difficulty: "Expert"
---

# How do you build a capacity plan that a SRE team can operationalize as autoscaling policies, and where do you deliberately keep humans in the loop instead of autoscaling a database?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Capacity Planning and Throughput](<../../../../01_Databases_and_SQL/Database_Operations/Capacity_Planning/Capacity_Planning_and_Throughput.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build an effective capacity plan for a database that SRE teams can operationalize into autoscaling policies, we need to ensure both technical and business considerations are well-balanced. The key is to define a scalable architecture with clear thresholds and triggers for human intervention where necessary. Let's break down the process from first principles.

#### Triaging the Question

The question asks about creating a capacity plan that not only supports autoscaling but also retains critical roles for humans in decision-making processes. Autoscaling policies are essential for maintaining performance, cost-efficiency, and reliability, but there should be boundaries where manual intervention is required to address more complex issues or avoid over-automation.

#### Assumptions
Database Environment: Assume we're dealing with a cloud-based RDBMS like AWS Aurora.
SRE Team Skills: The SRE team has the necessary skills in monitoring, alerting, and database management.
Business Goals: Maintain high availability while managing costs effectively.

#### First-Principles Solution

Define Metrics and KPIs:

CPU Utilization: High CPU usage indicates a potential need for more resources.
Read/Write Latency: Sluggish read/write operations suggest issues that could be resolved by scaling or tuning the database.
Disk I/O: Excessive disk I/O can indicate bottlenecks that require attention.

Capacity Modeling:

Use historical data to predict future load. Tools like AWS Budgets, Prometheus, and Grafana can help in collecting and visualizing metrics over time.
Estimate the database's performance under various workload scenarios using tools such as DyanmoDB Local or equivalent.

Threshold Definitions:

Establish thresholds for scaling: For example, if CPU usage exceeds 70%, initiate an autoscaling action to add more resources.
Define critical thresholds where human intervention is necessary: If the database goes into read/write lock, this requires immediate manual review and resolution.

Autoscaling Policies:

Implement autoscaling based on predefined metrics using cloud provider services like AWS Auto Scaling Groups.
Configure policies to automatically adjust resources within certain limits but ensure that thresholds are set high enough to avoid constant scaling actions.

Human In-the-Loop Mechanisms:

Complex Workloads: Certain workload patterns may require human oversight due to their complexity, such as backup operations or data migrations.
Critical Alerts: Set up alerting systems for critical conditions that could affect the database's availability and performance. For instance, if a major failure occurs, SREs should be notified immediately.

Regular Review and Optimization:

Continuously monitor the database’s performance and adjust scaling policies as needed.
Conduct periodic reviews to ensure that autoscaling is effective and human interventions are properly balanced.

Documentation and Training:

Document all threshold values, policies, and manual intervention steps for future reference.
Train SRE team members on how to handle both autoscaled actions and manual interventions effectively.

#### Closing Summary

By balancing automated scaling with human oversight, we can achieve a robust capacity plan that optimizes resource utilization while ensuring high availability. The key is to define clear thresholds and ensure the SRE team has the tools and knowledge to manage both auto-scaling actions and critical human decisions.

#### Common Mistakes/Follow-Ups
Over-Automation: Failing to set appropriate thresholds can lead to unnecessary scaling events, increasing costs.
Under-Supplying Resources: Not providing enough resources when needed can degrade performance.
Lack of Human Oversight: Relying too heavily on automation without human intervention mechanisms can result in missed critical issues.

Always validate the setup through real-world testing and feedback loops. Autoscaling policies should be fine-tuned based on actual usage patterns to ensure they align with business goals.

**Comparison:**

```
Metric	Threshold	Action Taken
CPU Usage	>70%	Autoscale to add instances
Read Latency	>120ms	Review and possibly optimize queries
Write Latency	>50ms	Investigate I/O bottlenecks
Disk I/O	>80%	Monitor for persistent issues
```

**Code:**

```python
# Example of a simple autoscaling policy using AWS SDK
import boto3

def auto_scale_instance(instance_id, new_size):
    ec2 = boto3.resource('ec2')
    instance = ec2.Instance(instance_id)

    # Scale up by adding instances
    if new_size > len([i for i in instance.instances.all()]):
        for _ in range(new_size - len([i for i in instance.instances.all()])):
            instance.create_instances(ImageId='ami-0abcdef1234567890', MinCount=1, MaxCount=1)

    # Scale down by removing instances
    elif new_size < len([i for i in instance.instances.all()]):
        instances_to_terminate = [i.id for i in instance.instances.all()]
        ec2.instances.filter(InstanceIds=instances_to_terminate).terminate()
```

**Example input / output:**

```text
Input:
- Metrics: CPU usage at 72%, read latency at 120ms, disk I/O at 85%
- Alerts triggered: None

Output:
- Autoscaling initiated to add one more instance due to high CPU and I/O usage.
- No immediate human intervention required as the thresholds have not been breached for critical alerts.
```

**Data:**

```json
{
  "metrics": {
    "cpu_usage_threshold": 70,
    "read_latency_threshold": 100ms,
    "write_latency_threshold": 50ms,
    "disk_io_threshold": 80%
  },
  "alerts": {
    "critical_failure_alerts": ["backup_failed", "major_service_interrupt"]
  }
}
```

**Flow:**

1. Yes
2. No
3. High
4. Low
5. High
6. Low
7. Yes
8. No
9. Monitor Metrics
10. CPU Usage?
11. Autoscale Instances
12. Check Read/Write Latency
13. Optimize Queries / Logs
14. Check Disk I/O
15. Monitor for Bottlenecks
16. Trigger Critical Alerts?
17. Human Review Required

