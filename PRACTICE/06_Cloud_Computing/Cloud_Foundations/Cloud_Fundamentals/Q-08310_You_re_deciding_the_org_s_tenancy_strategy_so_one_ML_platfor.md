---
type: "practice"
difficulty: "Expert"
---

# You're deciding the org's tenancy strategy so one ML platform serves both a cheap shared-inference tier and a premium single-tenant isolated tier, and can migrate tenants between them. How do you architect that boundary at the account, network, and accelerator layers?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Tenant Isolation](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Stateless_by_Default/Tenant_Isolation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we are tasked with designing an architecture that supports both a cheap shared-inference tier and a premium single-tenant isolated tier within a cloud environment. The key challenge is to ensure seamless migration between these tiers while maintaining strict security and performance requirements. This involves careful consideration at the account level (tenancy), network design, and accelerator layers.

#### Triage
Key Aspects Probed

The interviewer is probing your understanding of cloud fundamentals, specifically around tenancy models, network segmentation, and resource isolation. We need to ensure that tenants can be dynamically migrated between tiers without causing downtime or compromising security. Assumptions include the use of a managed cloud platform like AWS, GCP, or Azure.

#### First-Principles Solution
Account Level (Tenancy Model): Implement a hybrid tenancy model using organizational units (OUs) in AWS or equivalent structures in other clouds to isolate tenants.
Network Layer: Utilize Virtual Private Clouds (VPC) and Network Access Controls for segmentation, ensuring that resources are logically separated but still accessible when needed.
Accelerator Layers: Leverage accelerators like FPGAs or GPUs as necessary, with appropriate security groups and network policies to enforce isolation.
Detailed Solution
Account Level: Tenancy Model

We will use AWS Organizations to create a hierarchical structure of OUs for different tenants. Each OU can represent either the shared-inference tier or the premium single-tenant isolated tier. This allows us to apply different policies, roles, and permissions at each level:

Shared Inference Tier
B1
C1
Sub-Tenant 1
Sub-Tenant 2
Sub-Tenant 3
Sub-Tenant 4
Premium Single-Tenant Tier
H1
I1
Single-Tenant 1
Network Layer: VPCs and Network Policies

For the shared-inference tier, we will create a single VPC with multiple subnets. Each tenant in this tier will be assigned to specific subnets within this VPC. For the premium tier, each tenant will have their own dedicated VPC for complete isolation:

Shared Tier VPC
Subnet 1
Subnet 2
Premium Tier VPCs
Subnet 1 - Tenant 1
Subnet 2 - Tenant 2

Network Access Controls (NACs) and Network ACLs will be used to enforce traffic filtering, ensuring that only necessary traffic can pass between subnets or VPCs.

Accelerator Layers: GPU/Inference Management

For inference tasks, we will use managed services like AWS Elastic Inference for the shared tier. This allows us to scale GPU resources on-demand without provisioning physical hardware. For the premium tier, each tenant will have dedicated GPUs that are isolated from others:

Shared Tier
Elastic Inference
Managed ML Workloads
Premium Tier
Dedicated GPU Resources

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, our architecture leverages AWS Organizations for tenancy management, VPCs for network segmentation, and managed services like Elastic Inference to balance cost and performance. This approach ensures flexibility in tenant migration while maintaining security and compliance.

#### Common mistakes
include:

Overlooking the importance of proper role-based access control (RBAC) and least privilege.
Failing to implement robust logging and monitoring mechanisms.
Not considering edge cases, such as what happens during a network outage or when migrating tenants between tiers.

When asked follow-ups like "How would you handle data governance in this setup?" or "What about compliance with regulatory requirements?", it's essential to show your understanding of these aspects and how they fit into the overall architecture.

**Code:**

```python
# Example Code for Network Policies using Boto3 (AWS SDK)
import boto3

def create_vpc_policy(vpc_id, policy_name):
    ec2 = boto3.client('ec2')
    response = ec2.create_network_acl(
        VpcId=vpc_id,
        Tags=[
            {
                'Key': 'Name',
                'Value': policy_name
            },
        ]
    )
    return response['NetworkAcl']['NetworkAclId']
```

**Example input / output:**

```text
Input:
- Tenant requests migration from Shared Tier to Premium Tier

Output:
1. Tenant is reassigned to a new OU in the AWS Organization.
2. A new VPC and subnets are created for the tenant in the Premium Tier.
3. Existing resources are transferred or terminated as necessary.
4. Network policies are updated to reflect the new setup.
5. Access control roles and permissions are adjusted.
```

**Data:**

```json
{
  "Organization": {
    "Root": {
      "OU": [
        {"Shared Tier": ["Subnet1", "Subnet2"]},
        {"Premium Tier": ["Tenant1VPC", "Tenant2VPC"]}
      ]
    }
  },
  "Networks": {
    "SharedTierVPC": {
      "Subnets": ["Subnet1", "Subnet2"]
    },
    "PremiumTierVPC": {
      "Tenants": [
        {"Tenant1": "Subnet1"},
        {"Tenant2": "Subnet2"}
      ]
    }
  }
}
```

