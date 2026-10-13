---
type: "practice"
difficulty: "Expert"
---

# How do you build tiering into a self-serve data platform so domain teams get sane defaults but can't accidentally archive data that needs to stay hot?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Storage Tiering Design](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Storage_Tiering/Storage_Tiering_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Tiering in a self-serve data platform is essential for optimizing storage costs while ensuring critical data remains accessible and cost-effective. This solution involves creating a multi-tiered storage strategy with well-defined lifecycle policies, and leveraging automation to ensure that teams can manage their data without over-archiving.

First, we need to understand the key components of our tiering approach: Hot, Warm, and Cold tiers. Hot data is accessed frequently, Warm data is accessed occasionally, and Cold data is infrequently accessed but needs to be retained for regulatory or business reasons. Glacier serves as a deep storage tier with extremely low access patterns.

The problem here involves striking the right balance between cost optimization and data availability. Domain teams should have reasonable defaults that align with their use cases while being prevented from accidentally archiving critical hot data. Let's break this down step-by-step:

Define Storage Tiers:

Hot Tier (e.g., Amazon S3 Standard): Fast access, frequent usage.
Warm Tier (e.g., Amazon S3 Standard-Infrequent Access [S3 IA]): Occasional access with reduced cost compared to the standard tier.
Cold Tier (e.g., Amazon S3 One Zone-Infrequent Access [S3 One Zone IA] or S3 Glacier): Infrequent access, long-term storage.

Automated Lifecycle Policies:

Set up lifecycle policies that automatically move data between tiers based on usage patterns.
Use tags to categorize datasets and apply tiering rules accordingly.

Default Tiering Settings:

Provide default settings for new datasets that align with common use cases.
For example, newly uploaded data might be set as S3 Standard (hot) by default.

Access Control & Monitoring:

Implement fine-grained access controls to ensure only authorized personnel can change tiering policies or archive critical data.
Use monitoring tools to track data access patterns and alert on unusual activity that might indicate misuse of the archiving process.

Data Retention Policies:

Define strict retention policies for critical datasets to prevent accidental deletion or archival.
Use compliance and audit logs to enforce these policies across the platform.

User Education & Training:

Educate domain teams on best practices for tiering and lifecycle management.
Provide a user-friendly interface that simplifies policy creation and management.

**Comparison:**

```
Tier	Description	Storage Class	Cost
Hot	Frequently accessed	S3 Standard	High
Warm	Occasional access	S3 Standard-IA	Medium
Cold	Infrequent access, long-term storage	S3 One Zone IA or Glacier	Low
```

**Code:**

```python
# Example of setting up an S3 Lifecycle Policy in Python using Boto3

import boto3

s3 = boto3.client('s3')

lifecycle_config = {
    'Rules': [
        {
            'ID': 'HotTieringRule',
            'Prefix': 'hot/',
            'Status': 'Enabled',
            'Transitions': [
                {
                    'Days': 30,
                    'StorageClass': 'STANDARD_IA'
                }
            ],
            'NoncurrentVersionTransition': {
                'NoncurrentDays': 90,
                'StorageClass': 'GLACIER'
            },
            'ExpirationIn Days': 180
        }
    ]
}

s3.put_lifecycle_configuration(Bucket='example-bucket', LifecycleConfiguration=lifecycle_config)
```

**Example input / output:**

```text
Input:
- New data upload to S3 bucket 'example-bucket'
Output:
- Data automatically set to 'STANDARD' tier upon creation.
- After 30 days of inactivity, data moves to 'STANDARD_IA'.
- After 90 noncurrent versions (30 days * 3 months), data is archived to 'GLACIER'.
```

**Data:**

```json
{
  "Bucket": "example-bucket",
  "LifecycleConfiguration": {
    "Rules": [
      {
        "ID": "HotTieringRule",
        "Prefix": "hot/",
        "Status": "Enabled",
        "Transitions": [
          {
            "Days": 30,
            "StorageClass": "STANDARD_IA"
          }
        ],
        "NoncurrentVersionTransition": {
          "NoncurrentDays": 90,
          "StorageClass": "GLACIER"
        },
        "ExpirationInDays": 180
      }
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Data Upload
6. Is Data Hot?
7. Set to S3 Standard
8. Check Retention Policies
9. Archive to GLACIER or S3 OneZone IA
10. Keep in Warm Tier

