---
type: "practice"
difficulty: "Deep"
---

# How do you tier feature-store and training datasets when reproducibility demands you keep every historical snapshot but most are never re-read?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Storage Tiering Design](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Storage_Tiering/Storage_Tiering_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Tiering in a data engineering context, especially for feature stores and training datasets, is about balancing storage costs with the need for long-term archival while ensuring reproducibility of historical data. In this scenario, we need to keep every historical snapshot but minimize costs by moving less frequently accessed data to cheaper tiers like Glacier.

#### Triage

The question asks us to design a tiering strategy for feature stores and training datasets that balances storage cost with the requirement for retaining all historical snapshots. The key challenge is to avoid re-reading most of the cold (less-frequently accessed) data, which can be expensive in hot tiers, while ensuring that any required dataset version remains accessible.

#### First Principles Solution
Identify Historical Snapshots: Distinguish between recent and older snapshots.
Hot Tier Usage: Hot tier should store the latest or most frequently used datasets to ensure fast access for training and feature retrieval.
Cold Tier Storage: Store older but still valuable snapshots in a cold tier, which is less expensive than hot but slower to access.
Glacier Tier Archival: For data that hasn't been accessed in years, move it to the Glacier tier, where costs are minimal, even though access is more time-consuming.
Step-by-Step Solution:
1. Data Classification

Classify datasets based on their recency and frequency of use:

Hot Tier (e.g., S3 Standard): Store recent snapshots that are frequently accessed or required for current projects.
Cold Tier (e.g., S3 Intelligent-Tiering, S3 One Zone-Infrequent Access (S3 One Zone-IA)): Move older but still relevant data to this tier. This tier automatically optimizes storage costs by moving data between different tiers based on access patterns.
Glacier Tier (e.g., Amazon Glacier): Archive infrequently accessed or historical snapshots that are needed for compliance, auditing, or long-term preservation.
2. Automated Rules and Policies

Implement automated rules to move datasets between tiers:

Automated Transitions: Configure S3 Intelligent-Tiering policies to automatically transition data based on access patterns.
Manual Tags: Use tags to categorize datasets by their use cases (e.g., training, feature store) and lifecycle stages.
3. Cost Management

Ensure cost management through tier-aware queries:

Query Optimization: Modify database queries to prefer reading from hot tiers when possible. For example, if a query is expected to run frequently, ensure it uses data stored in the hot tier.
Cost-Effectiveness Metrics: Regularly review and adjust the tiering strategy based on cost-effectiveness metrics.
4. Data Retention

Implement robust retention policies:

Compliance and Audit Requirements: Ensure that historical snapshots are retained for compliance and audit purposes, even if they are infrequently accessed.
Data Governance: Develop a governance framework to oversee data lifecycle management and ensure alignment with organizational policies.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

#### Common Mistakes

Overlooking the cost implications of keeping all historical snapshots in hot tiers.
Failing to automate tier transitions, leading to manual overhead.
Ignoring compliance and audit requirements that necessitate retaining certain data.

#### Follow-Ups

How would you handle datasets with varying access patterns?
What are the security considerations when moving data between tiers?
How would you ensure seamless data retrieval from Glacier for long-term archival needs?

#### Triaging the Question

The question is probing your ability to design a storage tiering strategy that balances cost and accessibility while ensuring reproducibility. It requires an understanding of different storage tiers, automated policies, and cost management principles.

**Comparison:**

| Tier        | Description           | Example Storage Class     | Cost  | Access Time    |
|-------------|-----------------------|--------------------------|-------|----------------|
| Hot         | Fast access, recent   | S3 Standard              | High  | Seconds        |
| Cold        | Infrequent access     | S3 Intelligent-Tiering   | Medium | Hours         |
| Archive     | Long-term storage     | GLACIER                  | Low   | Days          |

**Code:**

```python
# Example S3 Intelligent-Tiering policy setup (Python Boto3)
import boto3

s3 = boto3.client('s3')

def configure_tiering(bucket_name):
    # Enable intelligent tiering on the bucket
    s3.put_bucket_lifecycle_configuration(
        Bucket=bucket_name,
        Configuration={
            'Rules': [
                {
                    'ID': 'Move data to Infrequent Access',
                    'Prefix': '',
                    'Status': 'Enabled',
                    'Transitions': [{
                        'Days': 30,
                        'StorageClass': 'STANDARD_IA'
                    }],
                    'NoncurrentVersionTransition': {
                        'NoncurrentDays': 90,
                        'StorageClass': 'GLACIER'
                    }
                },
                # Additional rules can be added as needed
            ]
        }
    )
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Feature Store - Hot Tier (Latest Data) | S3 Standard |
| Training Datasets - Cold Tier (Older but Relevant) | S3 Intelligent-Tiering |
| Historical Snapshots - Archive Tier (Long-Term Storage) | GLACIER |

**Data:**

```json
{
  "buckets": [
    {"name": "feature-store-hot", "tier": "S3 Standard"},
    {"name": "training-cold", "tier": "S3 Intelligent-Tiering"},
    {"name": "archive-glacier", "tier": "GLACIER"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Data Classification
6. Is the data frequentlyaccessed?
7. HOT Tier (S3 Standard)
8. Is the data older than 90 days?
9. COLD Tier (S3Intelligent-Tiering)
10. ARCHIVE Tier (GLACIER)

