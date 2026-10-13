---
type: "practice"
difficulty: "Deep"
---

# Explain the difference between VPC peering and a Transit Gateway. When does peering stop scaling and you move to a hub-and-spoke?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Network Devices and Layers](<../../../../06_Cloud_Computing/Cloud_Foundations/Networking/Network_Devices_and_Layers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this answer, we'll explore the differences between VPC peering and a Transit Gateway in AWS, when VPC peering reaches its limits, and when moving to a hub-and-spoke architecture becomes necessary.

TL;DR: VPC peering is straightforward for connecting two VPCs directly but scales poorly with many VPCs. A Transit Gateway (TGW) supports a more scalable, centralized approach by acting as an intermediary for routing traffic between multiple VPCs and on-premises networks. However, beyond a certain number of VPCs, the complexity and potential bottlenecks in TGWs may necessitate a hub-and-spoke architecture.

#### Triaging the Question
The question delves into the networking fundamentals of AWS, specifically comparing two methods for inter-VPC communication (VPC peering vs. Transit Gateway) and exploring when and why one might transition from VPC peering to a more complex setup like a hub-and-spoke architecture.

#### First-Principles Solution

VPC Peering Overview: VPC peering allows two VPCs within the same AWS Region to communicate directly as if they were subnets in the same network, using private IP addresses.
Transit Gateway (TGW) Overview: A TGW serves as a central connection point for multiple VPCs and on-premises networks. It supports routing between multiple routes and can manage traffic from different sources into a single VPC or direct it to other networks.

Transitioning from VPC Peering:

Limitations of VPC Peering:

Maximum Number of Peers (100): Each VPC can be peered with up to 100 other VPCs.
Complexity and Management Overhead: As the number of peers increases, managing static routes becomes cumbersome.

Use Cases for TGW:

Centralized Routing: A single TGW can route traffic from multiple VPCs in a scalable manner.
Enhanced Security: Provides better security controls compared to VPC peering by offering more granular routing policies and integration with VPC Flow Logs, AWS CloudTrail, etc.

Hub-and-Spoke Architecture:

When the number of VPCs exceeds what is practical for TGWs (e.g., over 50+ VPCs), a hub-and-spoke architecture becomes advantageous.
The central hub acts as the Transit Gateway, and spokes are the individual VPCs connected to it. This setup simplifies management and enhances scalability.

#### Closing Summary
While VPC peering is simpler and more straightforward for small-scale networks, its limitations in terms of maximum peers and complexity necessitate moving to a Transit Gateway or even a hub-and-spoke architecture as needs grow.

#### Common Mistakes/Follow-Ups

Mistake: Assuming that VPC peering can scale indefinitely without additional infrastructure.

#### Follow-Up
Understanding the specific use cases where a TGW is preferred over VPC peering, and knowing how to design a hub-and-spoke architecture effectively.

**Comparison:**

```
VPC Peering	Transit Gateway (TGW)	Hub-and-Spoke Architecture
Connects 2 VPCs directly	Centralized routing for multiple VPCs and on-premises networks	Simplifies management by centralizing routing through a single hub
Maximum of 100 peers	Enhanced security controls, better scalability	Hub as the central point with spokes as individual VPCs connected to it
```

**Code:**

```python
# Example Python code for creating a TGW in AWS using Boto3
import boto3

ec2 = boto3.client('ec2')

response = ec2.create_transit_gateway(
    Description='My Transit Gateway',
)

print(f"Transit Gateway created with ID: {response['TransitGateway']['TransitGatewayId']}")
```

**Example input / output:**

```text
Input:
Create a TGW and attach two VPCs to it.

Output:
Transit Gateway created with ID: tgw-1234567890abcdef0

VPC vpc-12345678 attached to TGW.

VPC vpc-23456789 attached to TGW.
```

**Data:**

```json
{
  "vpcs": [
    {
      "id": "vpc-12345678",
      "cidr_block": "10.0.0.0/16"
    },
    {
      "id": "vpc-23456789",
      "cidr_block": "10.1.0.0/16"
    }
  ],
  "tgw_id": "tgw-1234567890abcdef0"
}
```

**Flow:**

1. Yes
2. No
3. VPC Peering
4. Is your network small enough?
5. Directly connect using VPCpeering
6. HUB-and-Spoke Architecture
7. Hub (TGW) connects tomultiple Spokes (VPCs)

