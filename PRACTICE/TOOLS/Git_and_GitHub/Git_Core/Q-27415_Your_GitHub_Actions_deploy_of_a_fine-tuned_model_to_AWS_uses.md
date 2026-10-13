---
type: "practice"
difficulty: "Deep"
---

# Your GitHub Actions deploy of a fine-tuned model to AWS uses long-lived keys. Walk through migrating to OIDC federation and what ref/branch conditions in the trust policy stop a fork PR from assuming the role.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ OIDC Federation for Deploys](<../../../../TOOLS/Git_and_GitHub/Git_Core/OIDC_Federation_for_Deploys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the security concerns with long-lived keys in your GitHub Actions deploy of a fine-tuned model to AWS, we'll migrate to using OpenID Connect (OIDC) federation. This approach leverages Amazon Cognito and AWS Identity and Access Management (IAM) roles to securely manage temporary credentials for your deployments. The key is to configure the trust policy correctly so that only fork pull requests from certain branches can assume the required role.

In this solution, we'll break down the process into several steps:

Understand the Current Setup: Long-lived keys are risky as they are not rotated and can be exposed.
Design the OIDC Federation Architecture:
Use Amazon Cognito to manage identity providers (GitHub).
Configure an IAM role for your CI/CD pipeline with permissions to access AWS resources.
Implement the Trust Policy: Ensure that only specific branches in forked repositories can assume the IAM role.
Integrate GitHub Actions with OIDC Federation.

#### Triage

The question is asking how to migrate from using long-lived keys for deploying a model on AWS to an OIDC-federated approach, and specifically, what conditions should be placed in the trust policy of the IAM role to ensure that only specific branches in forked repositories can assume the role. The interviewer is likely probing your understanding of OIDC, GitHub Actions integration, and AWS IAM policies.

#### First-Principles Solution

Setup Amazon Cognito:

Create an identity pool with a federated provider set to GitHub.
Ensure that users in this identity pool have access to your repository by configuring the necessary authentication methods.

Configure IAM Role for GitHub Actions:

Create an IAM role with permissions to perform tasks required for deploying your model.
Attach policies and tags as needed, ensuring the role has minimal privileges.

Trust Policy for OIDC Federation:

Define a trust policy that allows only specific branches in forked repositories to assume this role.
JSON
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:cognito-idp:<region>:<account-id>:identitypool/<identity-pool-id>"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringLike": {
          "<github-oidc-provider>.sub": [
            "repo:<owner>/<repo>:refQRST/*",  // Replace with your actual owner/repo and branch pattern
            "org/<organization-name>:refQRST/*"  // Replace with your organization name if using org access
          ]
        }
      }
    }
  ]
}

GitHub Actions Workflow:

Set up a GitHub Actions workflow that uses the OIDC provider to assume the role.
Ensure the workflow only triggers on specific branches and forks.

#### Closing Summary
and Common Mistakes/Follow-Ups

By using OIDC federation, we can securely manage temporary credentials for our CI/CD pipeline without relying on long-lived keys. The trust policy ensures that only specific branches in forked repositories can assume the role, mitigating security risks. A common mistake is not properly configuring the StringLike condition to restrict access strictly.

#### Follow-up
questions might include:

How would you handle different authentication methods for GitHub Actions?
What additional steps would be needed if this were a multi-account AWS setup?
How do you manage key rotation in this scenario?

**Comparison:**

```
Condition	Description
repo:<owner>/<repo>:refQRST/*	Grants access for any pull request in the specified branches of a forked repository.
org/<organization-name>:refQRST/*	Grants access for any pull request in the specified branches within an organization.
```

**Code:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:cognito-idp:<region>:<account-id>:identitypool/<identity-pool-id>"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringLike": {
          "<github-oidc-provider>.sub": [
            "repo:<owner>/<repo>:refQRST/*",  // Replace with your actual owner/repo and branch pattern
            "org/<organization-name>:refQRST/*"  // Replace with your organization name if using org access
          ]
        }
      }
    }
  ]
}
```

**Example input / output:**

```text
name: Deploy Model

on:
  push:
    branches:
      - main
      - '*'
  pull_request:
    branches:
      - '*'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v2
      - name: Assume Role with OIDC
        uses: aws-actions/configure-aws-credentials@v1
        with:
          role-to-assume: "arn:aws:iam::<account-id>:role/<role-name>"
          external-id: "<external-id>"
          web-identity-token-file: ${{ secrets.GITHUB_IDENTITY_TOKEN }}
      - name: Deploy Model to AWS
        run: |
          # Your deployment commands here
```

**Data:**

```text
- Replace `<region>`, `<account-id>`, `<identity-pool-id>`, `<github-oidc-provider>`, `<owner>/<repo>`, and `<organization-name>` with your actual values.
```

**Flow:**

1. GitHub PR Trigger
2. GitHub Actions Workflow
3. Cognito IDP Authentication
4. AWS IAM Role Assumption
5. AWS Model Deployment

