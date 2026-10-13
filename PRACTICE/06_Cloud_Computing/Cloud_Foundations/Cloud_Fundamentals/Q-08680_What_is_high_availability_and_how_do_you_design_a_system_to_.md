---
type: "practice"
difficulty: "Deep"
---

# What is high availability and how do you design a system to achieve it in the cloud?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Availability Zone Resilience](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Availability_Zone_Resilience/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

High availability (HA) ensures that a system remains available during failures, whether those are hardware, software, or environmental issues. In this context, we'll focus on designing a backend system with high availability using cloud fundamentals. The key idea is to create redundant and isolated components that can failover seamlessly without affecting the overall service.

First, let's break down what the interviewer might be probing:

Understanding of HA concepts.
Knowledge of cloud services like AWS, GCP, or Azure.
Ability to design a robust and fault-tolerant system.

#### Cons
ideration of costs and trade-offs in HA designs.

Now, we will reason step by step from first principles:

Step 1: Define the Problem

We need a system that can handle any kind of failure (node failure, region failure, network issues) without losing availability to users. This involves designing a multi-layer architecture with redundancy at each layer.

Step 2: Architecture Overview

A typical high-availability architecture in the cloud will consist of:

Load Balancer: Distributes traffic across multiple servers.
Multi-AZ Deployment: Ensures that services are available even if one availability zone (AZ) goes down.
Database Replication: For stateful components, replication ensures data availability and can be fault-tolerant.
Auto Scaling: Ensures sufficient capacity to handle traffic spikes without manual intervention.
Step 3: Load Balancer

A load balancer should be deployed in a way that it can failover automatically. Using an Application Load Balancer (ALB) or Network Load Balancer (NLB), we can route traffic based on health checks and ensure that unhealthy instances are taken out of the rotation.

Load Balancer
AZ 1 - Node 1
AZ 1 - Node 2
AZ 2 - Node 1
AZ 2 - Node 2
Step 4: Multi-AZ Deployment

Deploying services across multiple availability zones ensures that if one AZ fails, the service continues to operate in another. For example, using AWS's VPC and Availability Zones (AZs) can ensure high availability.

Region 1 - AZ 1
Region 1 - AZ 2
Region 2 - AZ 1
Region 2 - AZ 2
Step 5: Database Replication

For databases, replication can be set up to ensure data availability. For example, using AWS Aurora Global Database or Azure SQL Always On Availability Groups.

Database
Global Read Replicas
R1
R2
Step 6: Auto Scaling

Auto-scaling ensures that the system can handle varying loads by dynamically adding or removing resources. This can be configured using cloud providers' auto-scaling services.

Scaling Group
Instance 1
Instance 2
Instance 3
Step 7: Monitoring and Alerting

Implementing monitoring and alerting mechanisms is crucial. Tools like CloudWatch or Stackdriver can be used to monitor the health of the system and trigger alerts for proactive maintenance.

Monitoring Service
A1
Load Balancer
Database

#### Closing Summary

High availability in a cloud environment is achieved by designing a multi-layered, redundant system that can handle failures gracefully. This involves using load balancers, multi-AZ deployments, database replication, auto-scaling, and robust monitoring. While this design ensures high availability, it comes with trade-offs such as increased costs and complexity.

#### Common Mistakes/Follow-Ups
Ignoring Edge Cases: Not considering all possible failure modes can lead to unexpected downtime.
Over-Engineering: Adding too many components without a clear need can increase costs and complexity unnecessarily.
Lack of Monitoring: Without proper monitoring, issues may go unnoticed until it’s too late.

**Comparison:**

```
Component	Type	Purpose
Load Balancer	AWS ALB	Distributes traffic across nodes.
Auto Scaling Group	AWS ASG	Manages node count based on demand.
```

**Code:**

```python
# Example of an AutoScaling group configuration in AWS CloudFormation template

Resources:
  MyAutoScalingGroup:
    Type: "AWS::AutoScaling::AutoScalingGroup"
    Properties:
      LaunchConfigurationName: !Ref MyLaunchConfig
      MinSize: '2'
      MaxSize: '4'
      AvailabilityZones:
        - "us-west-1a"
        - "us-west-1b"
```

**Example input / output:**

```text
Input:
- User requests a service from the load balancer.
Output:
- Load Balancer forwards request to one of the nodes in AZ 1, ensuring high availability and fault tolerance.
Dry-run Example:
User request -> ALB (Health Check) -> Node 1 in AZ 1 -> Service Response
```

**Data:**

```json
{
  "LoadBalancer": {
    "Type": "AWS::ElasticLoadBalancingV2::LoadBalancer",
    "Properties": {
      "Subnets": ["subnet-01234567890abcdef", "subnet-9876543210fedcba"],
      "AvailabilityZones": ["us-west-1a", "us-west-1b"]
    }
  },
  "AutoScalingGroup": {
    "Type": "AWS::AutoScaling::AutoScalingGroup",
    "Properties": {
      "LaunchConfigurationName": "MyLaunchConfig",
      "MinSize": 2,
      "MaxSize": 4,
      "AvailabilityZones": ["us-west-1a", "us-west-1b"]
    }
  }
}
```

**Flow:**

1. Unhealthy
2. Healthy
3. User Request
4. Load Balancer
5. Node Health
6. Failover to healthy Node
7. Service Response

