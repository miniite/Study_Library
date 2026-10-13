---
type: "practice"
difficulty: "Deep"
---

# Your GitHub Actions job that deploys updated alerting rules to your cloud monitoring backend uses long-lived API keys. Walk through migrating to OIDC federation, and what ref conditions in the trust policy prevent a fork PR from assuming the deploy role and pushing rogue rules.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ OIDC Federation for Deploys](<../../../../TOOLS/Git_and_GitHub/Git_Core/OIDC_Federation_for_Deploys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we are tasked with migrating our GitHub Actions job that deploys updated alerting rules to a cloud monitoring backend from using long-lived API keys to OIDC (OpenID Connect) federation. This approach enhances security by leveraging short-lived credentials and the least privilege principle. The trust policy of the IAM role used for deploying the alerting rules needs careful configuration to ensure only authorized pull requests can assume this role and push changes.

To achieve this, we will:

Migrate from long-lived API keys to OIDC.
Configure a GitHub App or a GitHub Actions runner with an OIDC token.
Update the trust policy of the IAM role used for deployment to allow only specific conditions based on the OIDC token.
Triaging the Problem

The core question here is about understanding how to migrate from using long-lived API keys to leveraging OIDC in a secure manner, specifically ensuring that fork pull requests cannot assume the deploy role. The trust policy of the IAM role must be correctly configured to enforce these conditions.

#### First-Principles Solution

GitHub App or GitHub Actions Runner Configuration:

Deploy a GitHub App or integrate an existing runner with OIDC.
Generate an OIDC token that will be used for authentication.

IAM Role Trust Policy:

Define the trust policy to restrict which subjects (users or roles) can assume this role.
Use conditions in the trust policy to ensure only authenticated users from specific repositories can assume the role.

OIDC Token Verification:

Ensure that the OIDC token issued by GitHub is verified against a known list of trusted tokens.
Detailed Steps
Step 1: GitHub App or Runner Setup

First, we will set up a GitHub App to issue OIDC tokens. This involves:

Creating a new GitHub App.
Generating an API token for the app to access repository information and trigger workflows.
Configuring the app to issue an OIDC token that can be used by our GitHub Actions workflow.
Step 2: Trust Policy Configuration

Next, we will configure the trust policy of the IAM role used for deploying alerting rules. The trust policy should look something like this:

JSON
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "<OIDC_ARN>"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "<IDENTIFIER-POD>:sub": ["github.com/user123"]
                }
            }
        }
    ]
}

Here, <OIDC_ARN> is the Amazon Resource Name of your OIDC provider in AWS. The StringEquals condition ensures that only users with a specific user ID (in this example, "user123") can assume the role.

Step 3: GitHub Actions Workflow

In our GitHub Actions workflow, we will use the OIDC token to authenticate and deploy changes. Here’s an example:

YAML
name: Deploy Alerting Rules

on:
  push:
    branches:
      - main

jobs:
  deploy-alerting-rules:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up AWS CLI
      uses: aws-actions/configure-aws-cliv2@v1
      with:
        region: us-west-2
        role-to-assume: arn:aws:iam::<ACCOUNT_ID>:role/deploy-role
        role-session-name: deploy-session

    - name: Deploy rules
      run: |
        aws cloudwatch put-metric-alarm \
          --alarm-name "ExampleAlarm" \
          --metric-name "CPUUtilization" \
          --namespace "AWS/EC2" \
          --statistic Average \
          --period 300 \
          --evaluation-periods 1 \
          --threshold 75.0 \
          --comparison-operator GreaterThanThreshold \
          --alarm-actions arn:aws:sns:<REGION>:<ACCOUNT_ID>:example-sns

#### Closing Summary
and Common Mistakes

By migrating to OIDC, we can ensure that only authenticated users from specific repositories can assume the deploy role. The trust policy must be carefully crafted to enforce these conditions, preventing unauthorized access.

#### Common Mistakes

Not verifying the OIDC token before assuming the IAM role.
Using overly permissive policies in the trust document.
Not properly configuring the GitHub App or runner to issue and use valid OIDC tokens.

**Code:**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::<ACCOUNT_ID>:oidc-provider/github.com"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "github.com/groups/user123": ["true"]
                }
            }
        }
    ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. GitHub Action
6. GitHub App Issuing OIDCToken?
7. Federated IDP
8. IAM Role with OIDC TrustPolicy
9. AWS API Call to Deploy Rules
10. Access Denied

