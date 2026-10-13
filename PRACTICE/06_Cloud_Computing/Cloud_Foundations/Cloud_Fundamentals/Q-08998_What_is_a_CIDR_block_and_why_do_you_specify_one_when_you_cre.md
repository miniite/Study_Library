---
type: "practice"
difficulty: "Basic"
---

# What is a CIDR block and why do you specify one when you create a VPC?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Private Endpoints and Network Paths](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Private_Endpoints_and_Network_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A CIDR block (Classless Inter-Domain Routing) is a way to define a range of IP addresses in a more efficient manner compared to using individual IP addresses. When creating a Virtual Private Cloud (VPC), specifying a CIDR block helps you allocate a specific range of IP addresses for the network, ensuring that it integrates seamlessly with other networks and resources.

To delve into why we specify a CIDR block when creating a VPC, let's break down the key aspects step by step:

Understanding CIDR Notation: CIDR notation uses a slash (/) to separate the IP address from the prefix length (e.g., 192.0.2.0/24). The prefix length indicates how many bits of the IP address are used for the network portion, and the remaining bits are used for host addresses.

Network Addressing: When you specify a CIDR block, you're defining a contiguous range of IP addresses that can be allocated to your VPC. For example, 192.0.2.0/24 defines an address space from 192.0.2.0 to 192.0.2.255.

Benefits and Considerations:

Efficiency: Using CIDR blocks allows for efficient allocation of IP addresses, reducing the need to manage individual IP assignments.
Scalability: You can specify larger or smaller address spaces as needed, making it easier to scale your VPC with future growth.
Security and Isolation: By defining a specific range, you ensure that only authorized traffic within this range is allowed, enhancing security.

Creating a VPC with CIDR Block:

When creating a new VPC in a cloud provider like AWS, Azure, or Google Cloud, you typically need to input a CIDR block.
For example, if you specify 192.0.2.0/28, the address space includes 16 usable IP addresses (from 192.0.2.4 to 192.0.2.15).

Subnets and CIDR Blocks: Within a VPC, subnets are created using smaller CIDR blocks taken from the main VPC block. This allows for further segmentation of the network into logical sub-units.

**Comparison:**

```
Network Address	Broadcast Address	Usable IP Addresses
192.0.2.0	192.0.2.15	16
```

**Code:**

```python
# Example Python code to create a VPC with a specified CIDR block in AWS using Boto3
import boto3

ec2 = boto3.resource('ec2')

vpc = ec2.create_vpc(CidrBlock='192.0.2.0/28')
print(f"VPC created with ID: {vpc.id}")
```

**Example input / output:**

```text
Input: Create a new VPC with the specified CIDR block.
Output:
- VPC ID: vpc-0123456789abcdef0
- CIDR Block: 192.0.2.0/28
```

**Data:**

```json
{
  "cidr_block": "192.0.2.0/28",
  "description": "Example CIDR block for a VPC"
}
```

**Flow:**

1. Create VPC
2. Specify CIDR Block
3. Yes
4. Create VPC with specified CIDRblock
5. No
6. Default CIDR block used

