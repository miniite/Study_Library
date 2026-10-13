---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you would use object storage as the backbone of an ML artifact registry so model versions are immutable, reproducible, and cheap to keep.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Object Storage Classes](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Object_Storage_Classes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: To use object storage effectively as the backbone of an ML artifact registry, we need to ensure immutability, reproducibility, and cost efficiency. We can achieve this by leveraging features like versioning, lifecycle management, and server-side encryption.

Core Concept: Object storage is ideal for storing immutable artifacts due to its inherent properties such as durability, scalability, and strong consistency guarantees. By setting up a robust system with object storage, we can maintain model versions that are always accessible, verifiable, and cost-effective.

To implement this solution, follow these steps:

Choose the Right Object Storage Service: Select a service provider like AWS S3, Google Cloud Storage, or Azure Blob Storage.
Versioning Strategy: Enable versioning to ensure every time you upload a model artifact, a new version is created without overwriting previous versions.
Lifecycle Management: Configure lifecycle policies to automate the cleanup of old artifacts based on your retention policy.
Server-Side Encryption: Protect data at rest using server-side encryption with keys managed by AWS KMS (AWS), Google Cloud KMS (Google Cloud), or Azure Key Vault (Azure).
Access Control and Logging: Implement fine-grained access controls, such as IAM roles in AWS, to restrict who can read or write to the storage. Enable logging for auditing purposes.
Cost Management: Monitor usage patterns and optimize costs through efficient use of storage tiers and automatic scaling.

By following these steps, we ensure that our ML artifacts are stored securely, remain immutable, and are easy to reference across different stages of model development and deployment.

#### Assumptions

The organization has a budget for cloud storage services.
There is a need to manage multiple versions of the same artifact.
Security and compliance requirements must be met.

Let's dive deeper into each step:

Choosing the Right Object Storage Service: For an ML registry, we should choose a service that offers high availability, low latency, and strong consistency. AWS S3, Google Cloud Storage, and Azure Blob Storage are good options due to their built-in redundancy and performance optimizations.

Versioning Strategy:

Enable Versioning: By enabling versioning on the object storage bucket, each upload creates a new version of the artifact.
Metadata Management: Store metadata such as model name, version number, creation timestamp, and associated hyperparameters to make it easier to track different versions.

Lifecycle Management:

Define Retention Policies: Set lifecycle policies to automatically delete or transition older versions based on a defined retention period.
Automated Cleanup: This helps in reducing storage costs by removing unused artifacts while maintaining access to historical versions when needed.

Server-Side Encryption:

Key Management: Use server-side encryption with AWS KMS, Google Cloud KMS, or Azure Key Vault to manage encryption keys securely.
Data Integrity: Ensure that the encrypted data can be decrypted only by authorized users who have access to the correct keys.

Access Control and Logging:

IAM Roles/IAM Policies: Implement strict IAM roles to control who can read/write/upload/download artifacts from the storage bucket.
Audit Logs: Enable logging for auditing purposes, ensuring that all actions are recorded and traceable.

Cost Management:

Storage Tiers: Use cost-effective storage tiers like infrequent access or archive storage classes if not every version needs frequent access.
Monitoring Tools: Utilize built-in monitoring tools provided by the cloud provider to track usage patterns and optimize costs over time.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

A common mistake is neglecting lifecycle management, leading to excessive storage costs or data loss due to accidental deletion of old versions.
Another pitfall is not properly securing access controls, which can lead to unauthorized access or breaches. Always ensure that only authorized personnel have the necessary permissions.

In summary, using object storage as an ML artifact registry backbone ensures immutability and reproducibility while keeping costs under control through careful planning and implementation of best practices.

**Comparison:**

```
Step	Description
1	Choose the right object storage service (e.g., AWS S3, Google Cloud Storage)
2	Enable versioning to maintain multiple artifact versions
3	Set lifecycle policies for automatic cleanup and cost optimization
4	Use server-side encryption with managed keys for data protection
5	Implement access controls using IAM roles and enable logging for audits
6	Monitor usage patterns and optimize costs by leveraging storage tiers
```

**Code:**

```python
# Example Python script to upload a model version with versioning enabled

import boto3

def upload_model_version(bucket_name, key, file_path):
    s3 = boto3.client('s3')

    # Ensure versioning is enabled on the S3 bucket
    response = s3.put_bucket_versioning(
        Bucket=bucket_name,
        VersioningConfiguration={
            'Status': 'Enabled'
        }
    )

    # Upload the model file to the S3 bucket with a version ID
    s3.upload_file(file_path, bucket_name, key)
```

**Example input / output:**

```text
Input:
- Upload a model version to the S3 bucket 'ml-artifact-registry' with key 'models/v1/model.h5'

Output:
{
    "VersionId": "v4f2h9873jkl09h7g6fg6ghj6fh789",
    "UploadTime": "2023-10-01T12:00:00Z"
}
```

**Data:**

```json
{
  "bucket_name": "ml-artifact-registry",
  "key": "models/v1/model.h5",
  "file_path": "/local/path/to/model.h5"
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Upload Model Version
6. Is Versioning Enabled?
7. Create New Version ID
8. Enable Versioning
9. Upload File to S3 Bucket withVersion ID
10. Check Lifecycle Policies
11. Are Old Versions Still Needed?
12. Do Not Delete Old Versions
13. Delete Old Versions

