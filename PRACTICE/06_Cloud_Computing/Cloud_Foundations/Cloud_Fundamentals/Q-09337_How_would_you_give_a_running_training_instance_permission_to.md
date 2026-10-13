---
type: "practice"
difficulty: "Intermediate"
---

# How would you give a running training instance permission to read from cloud storage without putting keys on the box?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Instance Roles and Access](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Instance_Roles_and_Access.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: To securely give a running training instance permission to read from cloud storage without exposing keys, we can use role-based access control (RBAC) and managed IAM policies within our cloud provider environment.

The key idea here is to leverage the security features provided by cloud services like AWS or GCP. By setting up appropriate roles and permissions through Identity and Access Management (IAM), we ensure that only authorized instances have access to specific resources, such as S3 buckets or Google Cloud Storage objects.

#### Triage
The question asks about securely granting a training instance permission to read from cloud storage without directly exposing keys like AWS Access Keys. This involves ensuring that the instance has the necessary permissions via an IAM policy while keeping sensitive information secure. We need to reason through the use of RBAC, managed policies, and possibly service accounts.

Step-by-Step Solution:

Identify the Storage Service: Determine whether we are using AWS S3 or Google Cloud Storage (GCS). The steps will be similar but with slight differences in terminology.

Create a Role or IAM Policy:

In AWS, create an IAM role specifically for training instances.
In GCP, define a role within the IAM system that grants read access to specific storage buckets.

Attach the Role to the Instance: Ensure the running instance is attached to the appropriate IAM role through its instance profile (AWS) or by assigning it as part of the VM setup process in GCP.

Verify Access Without Keys: Test the instance’s ability to read from the cloud storage using standard SDKs or APIs without exposing any keys.

#### Example
for AWS:

Create an IAM Role:

PLAINTEXT
aws iam create-role --role-name TrainingInstanceRole --assume-role-policy-document file://trust-policy.json

Attach S3 Read Permissions to the Role:

JSON
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": "arn:aws:s3:::<bucket-name>/*"
    }
  ]
}

Attach the Role to the Instance:

When launching an EC2 instance, attach the role via its launch configuration or by modifying the existing instance’s IAM role.

#### Example
for GCP:

Create a Service Account and Assign IAM Role:

BASH
gcloud iam service-accounts create training-instance-account --display-name "Training Instance"
BASH
gcloud projects add-iam-policy-binding <project-id> \
  --member="serviceAccount:<training-instance-account>@<project-id>.iam.gserviceaccount.com" \
  --role="roles/storage.objectViewer"

Assign the Service Account to a VM:

When creating or configuring the VM, use the service account as its identity.

#### Closing Summary
By using RBAC and managed IAM policies, we can ensure that our training instance has the necessary permissions without directly exposing keys, thereby enhancing security. This approach leverages built-in cloud provider features for secure access control.

#### Common Mistakes/Follow-Ups

Forgetting to create a trust policy (AWS) or service account (GCP).
Not properly attaching the role/policy to the instance.
Exposing sensitive information in code or configuration files.
Not regularly reviewing and updating IAM policies and roles as needed.

