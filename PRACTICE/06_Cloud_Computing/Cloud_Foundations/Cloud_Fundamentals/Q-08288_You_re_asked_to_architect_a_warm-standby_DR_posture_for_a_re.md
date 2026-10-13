---
type: "practice"
difficulty: "Expert"
---

# You're asked to architect a warm-standby DR posture for a revenue-critical real-time inference service across two cloud regions where GPU capacity differs. How do you meet an RTO without paying for a full idle GPU fleet in the standby region?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ GPU Quota Management](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/GPU_Sharing/GPU_Quota_Management.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a warm-standby disaster recovery (DR) posture for a revenue-critical real-time inference service across two cloud regions with differing GPU capacities, we need to ensure high availability while minimizing costs. The goal is to meet the Recovery Time Objective (RTO) without maintaining an idle and fully provisioned GPU fleet in the standby region.

Firstly, let's clarify what the question is probing:

Disaster Recovery Strategy: We need a plan that minimizes downtime in case of a failure.
Cost Optimization: The architecture should be cost-effective by avoiding unnecessary resource allocation.

#### Performance Considerations
Real-time inference services demand minimal latency and high performance.
Triaging the Problem

We will use a warm-standby approach, where the secondary region is actively monitored but not fully provisioned until needed. This involves setting up a mechanism to quickly scale resources in the standby region when the primary fails.

#### Assumptions
Differing GPU Capacities: The primary and secondary regions have different available GPU capacities.
Real-time Inference Service: High performance is critical, but it's acceptable to have some latency increase during failover.
Recovery Time Objective (RTO): The service needs to resume operations within a specific time frame.

#### Solution
from First Principles

Primary-Secondary Pairing:

Deploy the primary service in Region A with sufficient GPU capacity for peak loads.
In Region B, deploy a lightweight monitoring and failover system instead of full-scale services.

Monitoring and Health Checks:

Implement continuous health checks on the primary service to detect failures promptly.
Use cloud-native monitoring tools (e.g., AWS CloudWatch, Azure Monitor) to get early warning signals.

Automated Failover Mechanism:

Set up a failover script that can be triggered by health check failure notifications.
The failover script should automatically scale the necessary GPU resources in Region B and reroute traffic to the secondary service.

Cost-Efficient Scaling:

Utilize reserved instances or spot instances for the standby region to minimize costs when not in use.
Implement auto-scaling groups that can quickly increase resource allocation when failover occurs.

Traffic Management:

Use load balancers (e.g., AWS Elastic Load Balancer, Azure Application Gateway) to direct traffic to the primary service.
Set up routing policies to switch traffic to the secondary service in case of a failure detected in Region A.

Disaster Recovery Testing:

Regularly test the failover process to ensure reliability and performance.
Ensure that the failover does not introduce unacceptable latency or degradation in service quality.

#### Closing Summary

By implementing a warm-standby DR strategy with automated failover, continuous health monitoring, and cost-efficient scaling mechanisms, we can achieve an acceptable RTO while keeping costs low. This approach balances high availability with financial prudence by avoiding unnecessary GPU provisioning in the standby region until it’s needed.

#### Common Mistakes/Follow-Ups
Neglecting Cost Optimization: Failing to use cost-effective tools like reserved instances or auto-scaling.
Underestimating Failover Time: Not adequately testing failover processes, leading to prolonged downtime.
Ignoring Real-Time Performance: Overlooking the critical nature of real-time inference and allowing for excessive latency during failovers.

To address these concerns, thorough testing and iterative refinement are essential to ensure the solution meets both operational and financial objectives.

**Comparison:**

```
Region	Instance Type	GPU Capacity
us-west-2 (Primary)	p3.2xlarge	4 GPUs
eu-central-1 (Secondary)	t3.small	0 GPUs
```

**Code:**

```python
# Example Failover Script in Python (simplified)
import boto3

def scale_resources(region):
    ec2 = boto3.resource('ec2', region_name=region)
    instance_id = "i-0123456789abcdef0"  # Example instance ID
    ec2.create_instances(ImageId="ami-0abc1234567890example", MinCount=1, MaxCount=1)

def failover_to_secondary():
    print("Initiating failover to secondary region")
    scale_resources('region-b')
    update_load_balancer_target_group()

# Function to route traffic to the new primary
def update_load_balancer_target_group():
    elb = boto3.client('elbv2')
    target_group_arn = "arn:aws:elasticloadbalancing:us-west-2:123456789012:targetgroup/my-targets/abcdef123456"
    elb.modify_target_group_attributes(TargetGroupArn=target_group_arn, Attributes=[{'Key': 'Routinghttpcookieaffinity', 'Value': 'false'}])
```

**Example input / output:**

```

Input: Health check fails on primary service in us-west-2.

Output:

PLAINTEXT
Copy
Initiating failover to secondary region eu-central-1
Scaling up resources with instance type t3.small
Rerouting traffic to the new primary server
```

**Data:**

```json
{
  "primary_region": "us-west-2",
  "secondary_region": "eu-central-1",
  "instance_type_primary": "p3.2xlarge",
  "instance_type_secondary": "t3.small"
}
```

**Flow:**

1. Primary Service in us-west-2
2. Health Check
3. Failover Triggers
4. Scale Resources in eu-central-1
5. Reroute Traffic to Secondary

