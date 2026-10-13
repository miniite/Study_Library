---
type: "practice"
difficulty: "Deep"
---

# Your GitHub Actions deploy of a model to a SageMaker endpoint uses long-lived AWS keys. Walk through migrating to OIDC federation, and what ref/branch conditions in the trust policy prevent a fork PR from assuming the deploy role.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ OIDC Federation for Deploys](<../../../../TOOLS/Git_and_GitHub/Git_Core/OIDC_Federation_for_Deploys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're addressing the security concern of using long-lived AWS keys for deploying models to an Amazon SageMaker endpoint via GitHub Actions. The goal is to migrate to OIDC (OpenID Connect) federation and understand how ref/branch conditions in the trust policy can prevent unauthorized access from fork pull requests.

Firstly, let's break down what we're dealing with:

Long-lived AWS Keys: These are typically IAM user or root account keys that have broad permissions.
GitHub Actions: We use this to automate the deployment process triggered by events like push commits.
OIDC Federation: This allows for secure authentication and authorization through OpenID Connect, leveraging the identity provider's credentials.

The core idea is to set up a more secure deployment pipeline by using OIDC tokens instead of long-lived keys. Specifically, we'll configure an IAM role with an OIDC trust policy that restricts access based on ref/branch conditions in the Git repository.

#### Triage
and Assumptions
GitHub Actions Workflow: We assume a basic GitHub Actions workflow is already set up to trigger deployments.
SageMaker Endpoint: The endpoint will be managed by SageMaker, which we'll use as the target for our deployment.
IAM Role Configuration: We need an IAM role that can be assumed by the OIDC provider.
Step-by-Step Solution

Create a GitHub App and OIDC Provider:

First, create a new GitHub app with sufficient permissions to act as an OIDC provider.
The app will issue tokens for our deployments.

Configure IAM Role Trust Policy:

Create or update the trust policy of the existing IAM role that is used by the GitHub Actions workflow.
The trust policy should allow the OIDC provider to assume this role based on specific conditions.

Ref/Branch Conditions in Trust Policy:

Use ref/branch conditions within the trust policy to restrict which branches can trigger a role assumption.
This ensures that only pull requests from certain branches (e.g., main, staging) can initiate deployments, preventing unauthorized access from forks.

#### Example
of Trust Policy
JSON
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::<account-id>:oidc-provider/github.<repo>.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "github.com/external-id": "<github-actor-id>",
          "github.com/repo": "arn:aws:iam::<account-id>:oidc-provider/github.<repo>.com"
        },
        "ArnLike": {
          "aws:rds:source-identity": [
            "arn:aws:secretsmanager:<region>:<account-id>:secret:<branch-name>-OIDCProviderId-*",
            "arn:aws:secretsmanager:<region>:<account-id>:secret:<main-branch-name>-OIDCProviderId-*"
          ]
        },
        "Bool": {
          "github.com/event-type": ["push"]
        }
      }
    }
  ]
}
GitHub Actions Workflow Configuration:
Update the GitHub Actions workflow to use the OIDC token for authentication.
Ensure that the actions only run on approved branches.

#### Closing Summary
and Common Mistakes
The key takeaway is ensuring that our deployment process uses secure methods like OIDC federation with strict branch conditions.

#### Common mistakes
include:
Failing to restrict access based on ref/branch conditions, which can lead to unauthorized deployments from forks or pull requests.
Misconfiguring the trust policy, leading to incorrect role assumptions.

**Code:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::<account-id>:oidc-provider/github.<repo>.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "github.com/external-id": "<github-actor-id>",
          "github.com/repo": "arn:aws:iam::<account-id>:oidc-provider/github.<repo>.com"
        },
        "ArnLike": {
          "aws:rds:source-identity": [
            "arn:aws:secretsmanager:<region>:<account-id>:secret:<branch-name>-OIDCProviderId-*",
            "arn:aws:secretsmanager:<region>:<account-id>:secret:<main-branch-name>-OIDCProviderId-*"
          ]
        },
        "Bool": {
          "github.com/event-type": ["push"]
        }
      }
    }
  ]
}
```

**Example input / output:**

```text
steps:
  - name: Deploy Model to SageMaker
    env:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      MODEL_S3_URI: <model-s3-uri>
    run: |
      # Deployment script here
```

**Flow:**

1. GitHub Push Event
2. Federated Token Issued
3. IAM Role Assumed with OIDCToken
4. SageMaker Endpoint Deployed
5. Model S3 URI Retrieved andDeployed
6. Ref/Branch ConditionsEnforced
7. Avoids Unauthorized Fork PRs

