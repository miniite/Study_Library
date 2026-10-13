---
type: "practice"
difficulty: "Basic"
---

# What is IAM and how would you use a role to let your instance read documents from object storage for a RAG pipeline?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Identity and Access Foundations](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Identity_and_Access_Foundations/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview
In cloud environments, Identity and Access Management (IAM) is crucial for securing resources and controlling access based on roles and policies. For an AI engineer working with a Read-As-Generator (RAG) pipeline, understanding how to use IAM roles to grant access to object storage is essential.

#### Triage
The question asks about Identity and Access Management (IAM), specifically focusing on using roles to allow an instance to read documents from object storage. The interviewer might be probing your knowledge of IAM principles and how to configure policies effectively. Key assumptions include understanding the RAG pipeline's requirements and the structure of cloud services like AWS S3 or similar.

First Principles:

IAM Overview: IAM controls who can access resources in a cloud environment.
Roles: In AWS, for example, roles are essentially named sets of permissions that you can assign to entities such as EC2 instances, users, or other AWS services.
Access Control for RAG Pipeline: The goal is to ensure the instance can only read specific documents without full access to the entire storage.

#### Solution

Identify the Resources and Permissions Needed: For a RAG pipeline, you need read-only access to certain S3 buckets or objects.
Create an IAM Role: Define a role with minimum necessary permissions (e.g., s3:GetObject).
Attach Policies to the Role: Use policies like AmazonS3ReadOnlyAccess that grant read-only permissions on specific S3 resources.
Assign the Role to the Instance: Attach the role to your EC2 instance or other service that will run the RAG pipeline.

#### Example
Steps:

Step 1: Create an IAM role named RAGPipelineReader.
Step 2: Attach policies such as AmazonS3ReadOnlyAccess and a custom policy if needed.
Step 3: Configure your EC2 instance to use this role via the AWS Management Console, AWS CLI, or Terraform.

#### Closing Summary
By using IAM roles, you can finely control access to object storage resources. This approach ensures security while allowing necessary operations for the RAG pipeline. Common mistakes include granting overly broad permissions or failing to revoke unnecessary access when no longer needed.

#### Common Mistakes/Follow-Ups

Granting full admin-level access instead of minimal required permissions.
Failing to update or remove roles and policies when the application's requirements change.
Not regularly reviewing and auditing IAM policies for compliance and security.

**Comparison:**

```
Step	Action
1	Create IAM role RAGPipelineReader
2	Attach AmazonS3ReadOnlyAccess policy
3	Configure EC2 instance to use this role
```

**Code:**

```python
# Example Python code using Boto3 (AWS SDK)
import boto3

def attach_policy_to_role(role_name, policy_arn):
    iam = boto3.client('iam')
    response = iam.attach_role_policy(
        RoleName=role_name,
        PolicyArn=policy_arn
    )
    return response
```

**Example input / output:**

```text
Input:
- Create an IAM role named `RAGPipelineReader`.
- Attach the `AmazonS3ReadOnlyAccess` policy.
- Configure EC2 instance to use this role.

Output:
{
  "Role": {
    "RoleName": "RAGPipelineReader",
    "AssumeRolePolicyDocument": {...},
    "RoleArn": "arn:aws:iam::123456789012:role/RAGPipelineReader"
  },
  "Policies Attached": [
    {"PolicyArn": "arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"}
  ]
}
```

**Data:**

```
Role ARN: arn:aws:iam::123456789012:role/RAGPipelineReader
Policy ARNs:
arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
Custom policy:
JSON
Copy
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": ["arn:aws:s3:::your-bucket-name/*"]
    }
  ]
}
```

**Flow:**

1. Create IAM Role
2. Attach Policies
3. Configure Instance
4. Grant Access

