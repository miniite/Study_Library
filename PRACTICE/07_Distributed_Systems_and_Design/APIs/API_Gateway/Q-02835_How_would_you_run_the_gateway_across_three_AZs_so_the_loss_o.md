---
type: "practice"
difficulty: "Deep"
---

# How would you run the gateway across three AZs so the loss of one AZ doesn't drop more than a third of capacity, and what pre-provisioned headroom keeps the remaining two from tipping over?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Load Balancing and Zone Aware Routing](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Load_Balancing_and_Zone_Aware_Routing/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure high availability and reliability when running an API Gateway across three Availability Zones (AZs), we need to distribute traffic in such a way that losing one AZ reduces our capacity by no more than a third. This is typically achieved using a combination of load balancers, auto-scaling groups, and appropriate pre-provisioned headroom to ensure the remaining two AZs can handle increased traffic without overloading.

The core idea here is to leverage a multi-zone design that accounts for potential failures while maintaining performance. We'll break down this solution from first principles: understanding the problem, designing the architecture, implementing it with AWS services, and ensuring we have adequate headroom.

**Comparison:**

| Zone | Instance Count |
|------|---------------|
| us-west-2a | 1 |
| us-west-2b | 1 |
| us-west-2c | 1 |

**Code:**

```python
# Example code snippet for setting up an ASG with pre-provisioned capacity
import boto3

ec2 = boto3.client('ec2')
asg = boto3.client('autoscaling')

def create_autoscaling_group(zone_names):
    response = asg.create_auto_scaling_group(
        AutoScalingGroupName='api-gateway-asg',
        LaunchConfigurationName='lc-api-gateway',
        MinSize=1,  # Minimum instances in each AZ
        MaxSize=len(zone_names),  # Maximum is the number of zones
        DesiredCapacity=len(zone_names),  # Start with one instance per zone
        AvailabilityZones=zone_names,
        LoadBalancerNames=['lb-api-gateway'],  # Connect to load balancer
    )
    return response

def set_pre_provisioned_capacity(min_size, max_size):
    asg.put_scaling_policy(
        AutoScalingGroupName='api-gateway-asg',
        PolicyName='pre-provisioned-capacity',
        PolicyType='TargetTrackingScaling',
        TargetTrackingConfiguration={
            'PredefinedMetricSpecification': {
                'PredefinedMetricType': 'ASGAverageCPUUtilization',  # Example metric
            },
            'TargetValue': 0.5,  # Target CPU utilization (adjust as needed)
            'DisableScaleIn': True  # Ensure we don't scale in unexpectedly
        }
    )
```

**Example input / output:**

```text
Input:
- Zone names: us-west-2a, us-west-2b, us-west-2c

Output:
1. ASG is created with one instance in each zone.
2. Scaling policy configured to maintain a target CPU utilization of 50% across all instances.
3. Load balancer distributes traffic evenly among the three zones.

Dry-run:
- If one AZ fails (e.g., us-west-2a), remaining two AZs (us-west-2b and us-west-2c) handle increased load.
- Each zone has an instance, so losing one AZ reduces capacity by 1 out of 3 instances, or about a third.
```

**Data:**

```json
{
    "zone_names": ["us-west-2a", "us-west-2b", "us-west-2c"],
    "min_size_per_zone": 1,
    "max_size_per_zone": 3,  # Maximum of one instance per AZ
    "desired_capacity": 3,  # Start with one instance in each zone
    "load_balancer_name": "lb-api-gateway"
}
```

**Flow:**

1. Yes
2. No
3. User Request
4. Is Zone Down?
5. Redirect to Another AZ
6. Load Balancer DistributesTraffic
7. ASG Scales to MaintainCapacity
8. Traffic Handled by RemainingInstances

