---
type: "practice"
difficulty: "Basic"
---

# Explain how Elastic IP charges catch teams off guard. When do EIPs cost money, how do you find the leaking ones, and what's your cleanup approach?

**Difficulty:** Basic · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Shadow IT and Budget Evasion](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Shadow_IT_and_Budget_Evasion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Elastic IPs (EIP) in cloud environments like AWS can indeed surprise teams with unexpected costs due to their nature and common usage patterns. In this explanation, we will dive into how EIP charges work, identify when they cost money, discuss methods for detecting leaking EIPs, and outline a cleanup strategy.

TL;DR

Elastic IPs charge by being associated with an instance or having reserved time in the future, even if not currently attached to anything. To avoid unexpected costs, you should regularly audit your EIP allocation, monitor their association status, and periodically clean up unused ones.

#### Triage
What's Being Probed?

The question aims at understanding:

The mechanics of Elastic IP charges.
How to identify underutilized or leaked EIPs.
Effective strategies for managing and cleaning up these resources.

#### First Principles Solution
Understanding EIP Charges

Elastic IPs in AWS are public IP addresses that can be associated with an EC2 instance. When you allocate an EIP, it costs money unless it's reserved specifically for a time period (like 1 year or 3 years). The cost arises from the association of the EIP to something—typically an EC2 instance.

When Do EIPs Cost Money?

EIPs charge in two main scenarios:

When Associated with an Instance: As long as an EIP is associated with a running (or stopped) EC2 instance, you pay for it.
Reserved Time Period: If you have reserved the EIP for a year or three years, even if not currently attached to anything, you still incur costs during that reservation period.
Finding Leaking EIPs

To find leaking EIPs (those that are being charged but not providing value), follow these steps:

List All Elastic IPs: Use the AWS CLI or API to fetch a list of all your EIPs.
Filter by Association Status: Identify which ones are currently associated with an instance and which are unassociated.
Check Usage Logs: Review logs from instances that were previously attached to these EIPs, ensuring they no longer need them.
Evaluate Reserved Times: Check if any unattached EIPs have reserved time periods left.
Cleanup Approach
Disassociate Unnecessary EIPs: For those not currently associated with an instance, disassociate them and monitor for a period to confirm there’s no further use.
Release Unused EIPs: If they are completely unused and will not be re-associated soon, release the EIPs to free up costs.
Review Reserved Times: Evaluate if reserved times are still necessary or if extending them would save more than incurring additional charges.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By maintaining a regular audit of your Elastic IPs and taking proactive steps to disassociate and release unused ones, you can significantly reduce unexpected costs. Key mistakes include failing to regularly check association statuses, not monitoring usage logs, and over-reserving EIPs for future needs without clear justification.

#### Follow-up
questions might include:

How do other cloud providers handle similar charges?
What are the best practices for automating this process using DevOps tools?

**Code:**

```bash
# Example AWS CLI command to list all Elastic IPs
aws ec2 describe-addresses --query 'Addresses[*].[PublicIp, AllocationId]'
```

**Example input / output:**

| PublicIP | AllocationID   | AssociationStatus | InstanceID |
|----------|---------------|------------------|-----------|
| 54.321.65.10 | eipalloc-1234abcd | None             | -         |
| 54.322.78.20 | assoc-0987efcd   | AttachedTo: i-0123abcde | -         |

**Data:**

```json
{
  "ElasticIPs": [
    {
      "PublicIp": "54.321.65.10",
      "AllocationId": "eipalloc-1234abcd"
    },
    {
      "PublicIp": "54.322.78.20",
      "AssociationId": "assoc-0987efcd"
    }
  ]
}
```

