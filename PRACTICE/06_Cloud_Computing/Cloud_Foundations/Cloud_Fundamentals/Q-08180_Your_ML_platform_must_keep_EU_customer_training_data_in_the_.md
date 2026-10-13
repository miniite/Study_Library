---
type: "practice"
difficulty: "Deep"
---

# Your ML platform must keep EU customer training data in the EU. How would you architect storage, training compute, and model artifacts to enforce data residency?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Data Lifecycle and Retention](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Stateless_by_Default/Data_Lifecycle_and_Retention.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that EU customer training data is kept within the EU, we need a robust architectural design that encompasses both storage and computation aspects while adhering to stringent compliance requirements. The key components of this architecture include secure data storage, efficient model training compute resources, and seamless model artifact management.

#### Triage

The question probes how to architect an ML platform such that all customer training data is stored in the EU and processed by EU-based compute resources. We need to ensure data residency compliance while maintaining high performance and scalability. The assumptions include regulatory requirements like GDPR for EU customer data protection, which mandate that data must be stored within the EU.

#### First-Principles Solution
Data Storage:
Utilize cloud storage services compliant with EU regulations, such as Amazon S3 in a Frankfurt (EU) region or Azure Blob Storage in Europe.
Implement fine-grained access controls and encryption at rest to ensure data security.
Training Compute:
Deploy ML training workloads on compute resources hosted within the EU, using services like AWS EC2 instances in Frankfurt or Azure VMs in Germany.
Leverage managed machine learning services that support EU residency, such as Google AI Platform’s regions in Europe.
Model Artifacts Management:
Store trained models and artifacts in dedicated storage buckets within the EU.
Implement version control and lifecycle management for model artifacts to ensure they are stored in a compliant manner.

#### Closing Summary

By carefully selecting cloud services that offer data residency options, implementing robust security measures, and ensuring continuous compliance with regulatory requirements, we can build an ML platform that adheres to EU data protection laws while maintaining high performance and efficiency.

#### Common Mistakes/Follow-Ups
Mistake: Failing to use region-specific cloud services. Using global services without considering the location of underlying infrastructure could violate data residency rules.

#### Follow-Up
Question: How would you handle edge cases where the model artifacts need to be accessed from non-EU locations for testing or debugging purposes?

**Code:**

```python
# Example of setting up an S3 bucket in Frankfurt with encryption at rest
import boto3

s3 = boto3.client('s3', region_name='eu-central-1')

response = s3.create_bucket(
    Bucket='eu-training-data-bucket',
    CreateBucketConfiguration={
        'LocationConstraint': 'eu-central-1'
    }
)

print(response)
```

**Example input / output:**

```text
Input:
- Training data is uploaded to S3 bucket in Frankfurt.
- Model training tasks are scheduled on EC2 instances located in Frankfurt.

Output:
- Training data is encrypted at rest and stored securely within the EU.
- Model training processes run efficiently on EC2 instances, ensuring all compute resources are also hosted in the EU.
```

**Data:**

```json
{
  "bucket": "eu-training-data-bucket",
  "region": "eu-central-1",
  "encryption": {
    "SSE-S3": true,
    "KMSKeyID": "arn:aws:kms:eu-central-1:123456789012:key/abcd1234-abcd-1234-abcd-1234abcd1234"
  },
  "accessControls": [
    {
      "ID": "user-123",
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::123456789012:user/user-123"},
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": [
        "arn:aws:s3:::eu-training-data-bucket/*"
      ]
    }
  ]
}
```

