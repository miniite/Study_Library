---
type: "practice"
difficulty: "Deep"
---

# Your Python automation stores AWS credentials in a config file and a security review flagged it. Walk through migrating to IAM roles / instance profiles / IRSA and how boto3's credential provider chain resolves them.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Credential Handling](<../../../../TOOLS/Python/Python_Language/Credential_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to transition from storing AWS credentials in a config file using Python to utilizing IAM roles or Instance Profiles, which is more secure and aligns with best practices recommended by security reviews. The goal is to ensure that our application running on an EC2 instance can authenticate with AWS services without hardcoding credentials.

#### Triaging the Question

The question probes understanding of AWS Identity and Access Management (IAM) concepts like IAM roles and Instance Profiles, as well as the credential provider chain in boto3. We need to ensure that our Python application using boto3 is configured correctly to leverage these security features. The core assumptions are:

Our application runs on an EC2 instance.
The EC2 instance needs to access AWS resources such as S3, Lambda, etc.

#### First Principles Solution

IAM Roles or Instance Profiles:

IAM Role: An IAM role is a set of permissions that can be assumed by another entity (like an instance profile) for the purpose of temporary credentials.
Instance Profile: An instance profile associates an IAM role with an EC2 instance, allowing the instance to assume the role and gain access to AWS resources.

boto3 Credential Provider Chain:

boto3 automatically searches a set of credential providers in order until it finds valid credentials.
The default sequence is:
Environment variables (AWS_ACCESS_KEY, AWS_SECRET_KEY)
Shared configuration file (~/.aws/credentials)
IAM role for the instance (if running on EC2)
Step-by-Step Solution
Step 1: Define and Attach an IAM Role to Your Instance Profile

First, create an IAM role with appropriate policies attached that allow your application access to necessary AWS services.

BASH
aws iam create-role --role-name MyAppRole --assume-role-policy-document file://trust-policy.json

trust-policy.json should look something like this:

JSON
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {"Service": "ec2.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }
  ]
}

Then, create an instance profile and associate the IAM role with it:

BASH
aws iam create-instance-profile --instance-profile-name MyAppInstanceProfile

aws iam add-role-to-instance-profile --instance-profile-name MyAppInstanceProfile --role-name MyAppRole
Step 2: Ensure Your EC2 Instance is Running with the Correct Profile

Make sure your instance is using this instance profile by checking its metadata:

BASH
curl http://169.254.169.254/latest/meta-data/iam/info | jq '.'

This will return a JSON document indicating that the EC2 instance has assumed the IAM role.

Step 3: Configure boto3 to Use the Instance Profile

In your Python code, you can use the default credential provider chain which looks for temporary credentials:

PYTHON
import boto3

session = boto3.Session()
s3_client = session.client('s3')

#### Closing Summary
and Common Mistakes/Follow-Ups

By leveraging IAM roles and instance profiles, we ensure that our application does not hardcode sensitive credentials. The credential provider chain in boto3 will automatically fetch the temporary credentials provided by the EC2 instance's assumed role.

#### Common Mistakes

Forgetting to attach policies to the IAM role.
Not properly configuring the instance profile or associating it with the correct EC2 instances.
Hardcoding credentials directly into the code, which can lead to security breaches and compliance issues.

For follow-ups, we might discuss additional security measures like MFA (Multi-Factor Authentication) for assumed roles, or more advanced permissions management techniques such as AWS IAM Policies and Groups. Additionally, we could explore logging mechanisms to monitor who is assuming roles and when.

**Code:**

```python
import boto3

# Example usage of `boto3` with the default credential provider chain
session = boto3.Session()
s3_client = session.client('s3')
print(s3_client.list_buckets())
```

**Flow:**

1. Create IAM Role
2. Create Instance Profile
3. Attach Role to Profile
4. Run EC2 Instance with Profile
5. Check Metadata forCredentials
6. Use Default CredentialProvider Chain in `boto3`

