---
type: "practice"
difficulty: "Deep"
---

# Cross-AZ data transfer costs are ballooning. How do networking choices (topology, endpoints, replica placement) drive that cost and how do you cut it?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ DNS Failover](<../../../../06_Cloud_Computing/Cloud_Foundations/Networking/DNS_Failover.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a multi-zone AWS environment where cross-availability zone (AZ) data transfer costs are skyrocketing, understanding the underlying networking choices—such as topology design, endpoint selection, and replica placement—is crucial to optimizing both cost and performance.

#### Triaging the Question

The core of this question revolves around identifying how different network configurations impact the cost of data transfers between zones. The interviewer is probing your knowledge on how to architect a scalable and cost-effective network in AWS while ensuring minimal latency and high availability.

#### Assumptions
You are working with an AWS environment, specifically dealing with multiple AZs.
Cost efficiency is a primary concern.
High availability and data redundancy are necessary requirements.

#### First-Principles Solution

To minimize cross-AZ data transfer costs while maintaining performance and high availability:

Design the Topology:

Use a combination of Public Subnets for accessible resources, Private Subnets for private services, and VPC Peering to connect VPCs within different AZs.
Leverage AWS Direct Connect or a Network Load Balancer (NLB) to reduce the reliance on internet gateways.

Endpoint Selection:

Deploy stateless services in NLBs across multiple AZs to distribute traffic and minimize cold starts.
Utilize Amazon Route 53 for intelligent routing, which can automatically route requests based on latency or health checks.

Replica Placement:

Place replicas of critical data in each AZ to ensure redundancy and reduce cross-AZ transfer costs.
Implement an event-driven approach where updates are pushed to a central location, reducing the need for frequent cross-AZ syncs.

Cost Optimization Techniques:

Use AWS Lambda with API Gateway to handle lightweight workloads, reducing the need for constant data transfers.
Employ AWS Storage Gateway or S3 for offloading large datasets and archiving older data, minimizing cost-intensive transfers.

Monitoring and Cost Management:

Set up Amazon CloudWatch to monitor traffic patterns and identify inefficiencies.
Utilize AWS Budgets to set cost alerts and automatically scale resources based on usage.

#### Closing Summary

By carefully designing the network topology, selecting appropriate endpoints, placing replicas intelligently, and implementing cost optimization techniques, you can significantly reduce cross-AZ data transfer costs while maintaining high availability and performance. Common mistakes include over-reliance on internet gateways for inter-zone traffic, neglecting stateful services in NLBs, and not leveraging AWS managed services for cost savings.

#### Common Mistakes/Follow-Ups
Overlooking the use of VPC Peering or Direct Connect.
Not properly balancing NLB instances across AZs to avoid cold starts.
Failing to implement event-driven architectures where possible.
Ignoring the importance of monitoring and proactive cost management practices.

**Comparison:**

| Resource | AZ1 Subnet ID | AZ2 Subnet ID |
|----------|---------------|---------------|
| VPC Peering Connection | vpc-pcx-12345678 | vpc-pcx-abcdabcd |
| NLB in AZ1 | elb-12345678 | elb-abcdabcd |
| NLB in AZ2 | elb-efghijkl | elb-mnopqrst |

**Code:**

```python
# Example code for setting up an NLB in AWS
import boto3

session = boto3.Session(region_name='us-west-2')
elbv2 = session.client('elasticloadbalancingv2')

response = elbv2.create_load_balancer(
    Name='my-nlb',
    Subnets=['subnet-12345678', 'subnet-abcdabcd'],
    SecurityGroups=['sg-90abcdef']
)
print(response['LoadBalancerArn'])
```

**Example input / output:**

```text
Input:
- Deploy an NLB across AZs in a multi-VPC environment.

Output:
1. Create VPC Peering connections between the source and destination VPCs.
2. Set up NLBs in each AZ with targets spread across multiple subnets.
3. Route traffic through these NLBs using AWS Route 53 for load balancing and failover.
```

**Data:**

```json
{
  "vpcId": "vpc-12345678",
  "subnets": [
    { "id": "subnet-12345678", "az": "us-west-2a" },
    { "id": "subnet-abcdabcd", "az": "us-west-2b" }
  ],
  "securityGroups": [ "sg-90abcdef" ]
}
```

**Flow:**

1. Initial State
2. Create VPC Peering
3. Set up NLBs across AZs
4. Configure Route 53 forintelligent routing
5. Monitor and Optimize Costs

