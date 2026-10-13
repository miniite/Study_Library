---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you would choose an object storage class strategy for user-generated content that is hot for a week then rarely accessed, minimizing retrieval surprises.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Object Storage Classes](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Object_Storage_Classes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To manage user-generated content (UGC) that is hot for a week and then rarely accessed, we need to balance storage costs with access latency and availability requirements. The key is to choose an object storage class strategy that minimizes retrieval surprises while staying within budget constraints.

This question probes understanding of cloud storage classes, their characteristics like cost, durability, performance, and data lifecycle management. We'll assume a typical AWS S3 setup for this example but the principles apply broadly to other providers as well.

#### First-principles solution

Identify Hot Data: UGC that is accessed frequently in the first week should be stored in a high-performance class.
Tiering Strategy: Transition data from hot storage after one week into colder, less expensive classes.
Optimize Retrieval Latency: Ensure quick access to recently used content while allowing for slower retrieval of older data.

The full solution involves:

Choosing the Right Storage Class:
Use Standard or Standard-IA for initial week when access is frequent and predictable.
Automate Data Transition:
Set up lifecycle policies to automatically transition data after one week from Standard to One Zone-Infrequent Access (IA) or Glacier Deep Archive.
Monitor Costs and Performance:
Use AWS Cost Explorer and CloudWatch to monitor storage costs and access patterns.
Optimize Retrieval Paths:
Implement caching mechanisms like CDN (CloudFront) for frequently accessed content.

#### Closing summary
By leveraging cloud storage classes with automated lifecycle policies, we can ensure that the UGC is cost-efficiently stored while providing quick access to recently used content. Common mistakes include over-provisioning high-cost storage or failing to implement adequate data transition strategies.

**Comparison:**

```
Storage Class	Cost	Performance	Durability
Standard	High	Excellent	99.99%
Standard-IA	Medium	Good	99.99%
One Zone-IA	Low	Good	99.99%
Glacier Deep Archive	Very Low	Poor (7-10 days)	99.999999999%
```

**Code:**

```python
# Example Python code for setting up an S3 bucket with lifecycle policies using boto3

import boto3

def set_lifecycle_policy(bucket_name):
    s3 = boto3.client('s3')

    # Define the rules
    rule1 = {
        'ID': 'Transition to One Zone-IA after 7 days',
        'Status': 'Enabled',
        'Transitions': [
            {'Days': 7, 'StorageClass': 'ONEZONE_IA'}
        ],
        'NoncurrentVersionTransition': None,
        'NoncurrentDays': 0
    }

    # Create the lifecycle configuration
    lifecycle_config = {
        'Rules': [rule1]
    }

    # Apply the policy to the bucket
    s3.put_bucket_lifecycle_configuration(
        Bucket=bucket_name,
        LifecycleConfiguration=lifecycle_config
    )
```

**Example input / output:**

```text
Input:
- UGC content uploaded to S3 bucket with initial storage class set as Standard

Output:
- After one week, the lifecycle policy transitions the data to One Zone-IA automatically

Request:
s3.put_object(Bucket='my-ugc-bucket', Key='user-uploads/12345.png')

Response (log from AWS CloudWatch):
"Lifecycle transition triggered for object 'user-uploads/12345.png' after 7 days."
```

**Data:**

```json
{
  "Bucket": "my-ugc-bucket",
  "LifecycleRules": [
    {
      "ID": "Transition to One Zone-IA after 7 days",
      "Status": "Enabled",
      "Prefix": "",
      "ExpirationInDays": 0,
      "Transitions": [
        {
          "Days": 7,
          "StorageClass": "ONEZONE_IA"
        }
      ],
      "NoncurrentVersionTransition": null
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. User Uploads UGC
6. Is Content Hot?
7. Store in Standard
8. Has One Week Passed?
9. Transition to One Zone-IA
10. Monitor and Optimize

