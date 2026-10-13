---
type: "practice"
difficulty: "Expert"
---

# A dataset is under legal hold but your lifecycle rule tried to expire it. How do you architect lifecycle so legal holds and retention locks always win over cost policies?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Storage Tiering Design](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Storage_Tiering/Storage_Tiering_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

In this scenario, we need to design a storage tiering and lifecycle management system that respects legal holds and retention locks while considering cost optimization. The solution involves leveraging metadata tags, customized lifecycle policies, and automated governance rules to ensure compliance without compromising on costs.

#### Triage

The core question is about architecting a storage system where the lifecycle management policies must yield to specific legal requirements. This means that when a dataset is under a legal hold or retention lock, the system should not trigger any actions that would violate these constraints, such as moving data to cheaper storage tiers or expiring it.

#### Assumptions
The storage system supports lifecycle rules based on age and metadata tags.
Legal holds and retention locks are managed via metadata tags.
Cost optimization policies can be defined independently of compliance requirements.
There is a mechanism for governance rules that override standard lifecycle actions when necessary.

#### First-Principles Solution
Metadata Management: Implement or use existing mechanisms to tag datasets with legal hold statuses and retention periods. These tags will serve as the primary indicator for lifecycle policies.
Governance Rules: Define custom governance rules that check these metadata tags before applying any lifecycle actions. If a dataset is under legal hold, ensure it remains in its current tier until the hold is lifted.
Lifecycle Policies: Design standard lifecycle policies based on age and usage patterns, but configure them to be overridden by the governance rules when necessary.
Step-by-Step Solution:

Tagging Datasets:

Use S3 Object Tags or equivalent metadata tagging in your storage solution (e.g., AWS S3).
For example, tag datasets with legal_hold and retention_period.

Custom Governance Rules:

Create a governance rule that checks the tags before any lifecycle action.
PYTHON
def check_legal_hold(bucket_name, object_key):
    s3 = boto3.client('s3')
    response = s3.get_object_tagging(Bucket=bucket_name, Key=object_key)
    for tag in response['TagSet']:
        if tag['Key'] == 'legal_hold' and tag['Value'] == 'true':
            return True
    return False

Lifecycle Policies:

Define lifecycle policies that consider age and usage patterns.
YAML
Rules:
  - ID: legal-hold-skip-expiration
    Status: Enabled
    Prefix: legal/
    Expiration: Disabled  # Skip expiration for legal hold datasets
  - ID: auto-tiering
    Status: Enabled
    NoncurrentVersionExpiration: 365  # Tier non-current versions after a year

Enforcement:

Ensure the governance rules are applied before any lifecycle policy enforcement.
PYTHON
def enforce_lifecycle_policy(bucket_name, object_key):
    if check_legal_hold(bucket_name, object_key):
        return  # Do not apply lifecycle actions
    else:
        s3 = boto3.client('s3')
        response = s3.get_bucket_lifecycle_configuration(Bucket=bucket_name)
        for rule in response['Rules']:
            if 'Prefix' in rule and rule['Prefix'] == object_key[:len(rule['Prefix'])]:
                apply_lifecycle_actions(s3, bucket_name, object_key, rule)  # Apply standard actions

#### Closing Summary

By implementing metadata tags to indicate legal holds and retention periods, custom governance rules that check these tags, and lifecycle policies configured to respect these constraints, we ensure compliance with legal requirements while still optimizing costs where possible. This approach requires careful planning and testing to ensure correctness.

#### Common Mistakes/Follow-Ups
Forgetting to Apply Governance Rules: Ensure the metadata checks are always performed before applying lifecycle actions.
Improper Tagging Practices: Incorrect tagging can lead to violations of legal holds or retention locks.
Overlooked Retention Policies: Ignoring specific retention periods defined by legal requirements.

