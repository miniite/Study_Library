---
type: "practice"
difficulty: "Intermediate"
---

# How do you handle snapshot and AMI sprawl that quietly grows storage cost? Describe the lifecycle automation you'd build and the safety checks before deletion.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Retention Costs and Compliance](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Retention_Costs_and_Compliance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: To address snapshot and AMI sprawl in FinOps, we'll implement a comprehensive lifecycle management (LCM) system that includes automatic tagging, retention policies, and pre-deletion safety checks. This ensures cost optimization while maintaining compliance and availability.

#### Triage
The question is asking how to manage the growth of storage costs due to excessive snapshots and AMIs by automating their lifecycle and ensuring responsible deletion practices. The core concepts include tagging for visibility, setting up policies based on age or size, and implementing checks before deletion to prevent accidental removal of critical resources.

#### First Principles Solution

Tagging and Labeling: Tag each snapshot and AMI with metadata such as environment (dev, prod), application name, creation date, and owner.
Retention Policies: Define policies based on age or size. For example, retain production snapshots for 30 days but only for critical applications, while dev environments can have stricter limits like 7 days.
Safety Checks Before Deletion:
Verify if the AMI or snapshot is currently in use (e.g., attached to an active instance).
Check if there are any open tickets or dependencies associated with it.
Ensure compliance with organizational policies.

#### Implementation
Steps:

Tagging System: Develop a tagging system that automatically applies relevant labels at creation time and updates them as needed.
Policy Engine: Build a policy engine that enforces retention rules based on tags and other metadata.
Notification Mechanisms: Set up notifications to alert stakeholders before deletion, ensuring they have enough time to review and approve deletions.
Automation Scripts: Write scripts to automate the tagging process and enforce policies during regular clean-up operations.

#### Closing Summary
& Common Mistakes:

This approach ensures that storage costs are optimized by eliminating unnecessary AMIs and snapshots without disrupting production or dev environments.
Key mistakes include over-reliance on manual processes, lack of proper tagging strategies, and insufficient safety checks leading to accidental deletions.

**Comparison:**

```
Environment	Snapshot Retention Type	Age/Size
Prod	Age	30 days
Dev	Size	< 5GB
```

**Code:**

```python
# Example Python script for tagging resources with Boto3 (AWS SDK)
import boto3

def tag_resource(resource_id, tags):
    ec2 = boto3.client('ec2')
    response = ec2.create_tags(
        Resources=[resource_id],
        Tags=tags
    )
    print(f"Tags applied: {response}")

# Example of tagging a snapshot
tag_resource("snap-12345678", [{"Key": "Environment", "Value": "Dev"}, {"Key": "Owner", "Value": "UserA"}])
```

**Example input / output:**

```text
Input:
- AMI ID: ami-1234567890abcdef0
- Snapshot ID: snap-0123456789abcdef

Output:
- Tagging Action: Tags applied to [AMI ID] and [Snapshot ID]
- Policy Evaluation: Retention policy matched, no deletion action taken.

Input:
- AMI ID: ami-01234567890abcdef
- Snapshot ID: snap-1234567890abcdef

Output:
- Tagging Action: Tags applied to [AMI ID] and [Snapshot ID]
- Policy Evaluation: Retention policy not matched, AMI eligible for deletion.
- Safety Check: AMI in use by instance i-1234567890abcdef -> Deletion blocked.
```

**Data:**

```yaml
# Sample retention policy configuration
retention_policies:
  - name: Production_Snapshots
    type: Age
    value: 30
    environments: [Prod]
  - name: Development_Snapshots
    type: Size
    value: 5GB
    environments: [Dev, Test]
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Tagging and Labeling
6. Retention Policy Evaluation
7. In Use?
8. Notify Owner
9. Pre-Deletion Safety Check
10. Compliant?
11. Delete Resource
12. Block Deletion

