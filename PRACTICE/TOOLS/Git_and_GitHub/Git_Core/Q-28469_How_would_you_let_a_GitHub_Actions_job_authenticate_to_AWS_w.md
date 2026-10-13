---
type: "practice"
difficulty: "Deep"
---

# How would you let a GitHub Actions job authenticate to AWS without storing long-lived access keys as secrets?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ OIDC Federation for Deploys](<../../../../TOOLS/Git_and_GitHub/Git_Core/OIDC_Federation_for_Deploys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To securely authenticate a GitHub Actions job to AWS without using long-lived access keys, we need to leverage temporary credentials that are short-lived and can be revoked or refreshed as needed. This approach minimizes the risk associated with exposing sensitive information like access keys. We'll use AWS IAM roles with Amazon EC2 instance profiles (for EC2 instances) or by using aws-iam-token in GitHub Actions.

TL;DR
Use AWS IAM Roles for EC2: For jobs running on an EC2 instance, attach a role that allows the instance to assume a trusted role.
GitHub Actions with AWS IAM: For jobs running directly from GitHub Actions, use the aws-iam-token action or similar mechanisms to retrieve temporary credentials.

#### Triaging the Question

The core of this question is about securely managing and accessing AWS resources via GitHub Actions without compromising security by storing long-lived credentials. The interviewer might be probing your understanding of best practices in DevSecOps and how to leverage AWS Identity and Access Management (IAM) effectively.

#### Solution
from First Principles
Using IAM Roles for EC2 Instances
Create a Trusted Role: Define an IAM role that trusts the instance profile associated with the EC2 instance.
Attach Permissions Policies: Grant this role the necessary permissions to access AWS resources like S3, Lambda, etc.
Instance Profile Assignment: Attach the created IAM role to the EC2 instance via its instance profile.

GitHub Actions Workflow Example:

YAML
name: Deploy

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          role-to-assume: arn:aws:iam::<account-id>:role/<trusted-role-name>
          role-session-name: GitHubActionsSession
      - name: Deploy application
        run: |
          # Commands to deploy the application using AWS CLI or SDKs
Using aws-iam-token in GitHub Actions
Install the aws-actions/configure-aws-credentials Action: This action can assume an IAM role and retrieve temporary credentials.
Specify the Role ARN: Provide the ARN of the IAM role that you want to assume.

GitHub Actions Workflow Example:

YAML
name: Deploy

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Configure AWS credentials with IAM role
        id: configure-aws-credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          role-to-assume: arn:aws:iam::<account-id>:role/<trusted-role-name>
          role-session-name: GitHubActionsSession
      - name: Deploy application
        run: |
          # Commands to deploy the application using AWS CLI or SDKs

#### Closing Summary
and Common Mistakes/Follow-Ups
Common Mistake: Storing long-lived access keys in secrets directly.

#### Follow-Up
Discuss additional security measures like MFA, least privilege principles, and monitoring for unauthorized access.

Using temporary credentials is a best practice that minimizes risk by ensuring that even if the token is compromised, it will expire soon. This method aligns with DevSecOps principles and ensures secure and scalable AWS integration within GitHub Actions.

**Code:**

```yaml
# Example of configuring AWS credentials in GitHub Actions using an assumed role

name: Deploy

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Configure AWS credentials with IAM role
        id: configure-aws-credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          role-to-assume: arn:aws:iam::<account-id>:role/<trusted-role-name>
          role-session-name: GitHubActionsSession
      - name: Deploy application
        run: |
          # Commands to deploy the application using AWS CLI or SDKs
```

