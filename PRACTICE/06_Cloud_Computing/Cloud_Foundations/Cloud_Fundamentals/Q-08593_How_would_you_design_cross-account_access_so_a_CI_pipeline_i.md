---
type: "practice"
difficulty: "Deep"
---

# How would you design cross-account access so a CI pipeline in one cloud account can deploy to production in another without long-lived keys?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Instance Roles and Access](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Instance_Roles_and_Access.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing cross-account access for a CI (Continuous Integration) pipeline, we need to ensure that sensitive information such as API keys and secrets are handled securely while still allowing necessary actions across different AWS accounts. The goal is to avoid using long-lived credentials by leveraging mechanisms like IAM roles with temporary security tokens.

To achieve this, the core idea involves using AWS Identity and Access Management (IAM) policies and roles, specifically focusing on Cross-Account Roles and Temporary Security Tokens via STS (Security Token Service). This approach ensures that your CI pipeline can securely assume an IAM role in a different account to perform deployments without needing long-lived keys.

#### Triage
The question is asking how to set up secure cross-account access for a CI pipeline so it can deploy resources into another AWS account. The interviewer may be probing your understanding of IAM policies, roles, and STS, as well as your ability to design secure, scalable solutions. We will make several assumptions: you are working with two AWS accounts (Account A: CI pipeline, Account B: Production), and both accounts have the necessary permissions and configurations.

Step-by-Step Solution:

IAM Role in Target Account: Create a dedicated IAM role (e.g., ProdDeployRole) in the target account (Account B) that has permission to perform the required deployment tasks.
Trust Policy for Cross-Account Access: Define an IAM policy on the ProdDeployRole with a trust relationship that allows the source account (Account A) to assume this role via STS credentials.
Temporary Security Tokens from Source Account: In the CI pipeline, use AWS SDKs or CLI tools to assume the ProdDeployRole in Account B using an IAM user or role from Account A as the principal. This will generate temporary security tokens that can be used for the deployment process.

#### First-Principles Solution

IAM Role Creation: In Account B (Production), create a new IAM role with the necessary permissions to deploy resources, e.g., ProdDeployRole.

JSON
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": ["arn:aws:s3:::example-bucket"]
    },
    {
      "Effect": "Allow",
      "Action": ["ec2:RunInstances", "cloudformation:CreateStack"],
      "Resource": "*"
    }
  ]
}

Trust Policy Configuration: In the ProdDeployRole, add a trust policy that allows users from Account A to assume this role.

JSON
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::ACCOUNT_A_ID:user/deployer"},
      "Action": ["sts:AssumeRole"]
    }
  ]
}

Temporary Security Tokens: In the CI pipeline (Account A), use the AWS SDK or CLI to assume the ProdDeployRole in Account B and generate temporary security tokens.

BASH
aws sts assume-role \
  --role-arn arn:aws:iam::ACCOUNT_B_ID:role/ProdDeployRole \
  --role-session-name ProdSession \
  --duration-seconds 3600 # Set the session duration as needed

Deployment Execution: Use these temporary credentials to execute deployment tasks in Account B.

BASH
aws s3 cp local-bucket s3://example-bucket --profile ProdDeployRole-credentials
aws ec2 run-instances --image-id ami-0c55b108 --key-name my-key-pair --security-group-ids sg-0123456789abcdef0 --instance-type t2.micro --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=DeployedByPipeline}]"

#### Closing Summary
+ Common Mistakes/Follow-Ups: This approach ensures secure and efficient cross-account access by utilizing temporary security tokens. It avoids the risks associated with long-lived credentials while still allowing necessary actions to be performed. Key best practices include regular review of IAM policies, careful management of session durations, and ensuring that all sensitive information is handled securely.

#### Common mistakes
might include not configuring proper trust relationships or using overly permissive policies. Also, failing to rotate temporary security tokens regularly could introduce vulnerabilities.

**Comparison:**

|Step| Action|
|---|---|
|1| Create ProdDeployRole in Account B with necessary permissions|
|2| Add trust policy to allow cross-account role assumption|
|3| Assume ProdDeployRole from CI pipeline using temporary STS credentials|

**Code:**

```bash
# Example AWS CLI command to assume a role in another account
aws sts assume-role \
  --role-arn arn:aws:iam::ACCOUNT_B_ID:role/ProdDeployRole \
  --role-session-name ProdSession \
  --duration-seconds 3600 # Set the session duration as needed

# Example usage of temporary credentials
aws s3 cp local-bucket s3://example-bucket --profile ProdDeployRole-credentials
```

**Data:**

```json
{
  "ProdDeployRole": {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": ["s3:ListBucket"],
        "Resource": ["arn:aws:s3:::example-bucket"]
      },
      {
        "Effect": "Allow",
        "Action": ["ec2:RunInstances", "cloudformation:CreateStack"],
        "Resource": "*"
      }
    ]
  },
  "TrustPolicy": {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Principal": {"AWS": "arn:aws:iam::ACCOUNT_A_ID:user/deployer"},
        "Action": ["sts:AssumeRole"]
      }
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. CI Pipeline (Account A)
4. Assume Role in Account B
5. Execute Deployment
6. Complete Deployment
7. Review IAM Policies, RotateCredentials

