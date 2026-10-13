---
type: "practice"
difficulty: "Deep"
---

# Your GitHub Actions job that applies migrations to a staging RDS uses long-lived AWS keys. Walk through migrating to OIDC federation and what ref/branch conditions in the trust policy stop a fork PR from assuming the deploy role.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ OIDC Federation for Deploys](<../../../../TOOLS/Git_and_GitHub/Git_Core/OIDC_Federation_for_Deploys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to transition from using long-lived AWS access keys to an OIDC (OpenID Connect) federated identity model with GitHub Actions for our staging RDS migration jobs. This change ensures that our deployment process is more secure by leveraging temporary credentials and reducing the risk of sensitive key exposure.

#### Triage

The question asks us to:

Understand how OIDC federation can replace long-lived AWS keys.
Detail the necessary changes in trust policies to restrict which branches or forks can assume a specific IAM role used for deployments.

#### First-Principles Solution
Step 1: Create an OIDC Provider and Client

First, we need to set up an OpenID Connect (OIDC) provider within AWS Identity and Access Management (IAM). This involves creating an OIDC identity provider that connects to GitHub's OAuth endpoints. Once the OIDC provider is created, we can create a client for it in IAM.

BASH
aws iam create-open-id-connect-provider --url <GitHub App OAuth URL>
aws iam create-client-id-for-open-id-connect-provider --open-id-connect-provider-arn arn:aws:iam::<AccountID>:oidc-provider/<ProviderName> --client-name git-actions
Step 2: Update the GitHub Actions Workflow

Next, we update our GitHub Actions workflow to use the OIDC client. This involves using an actions/identity step from the GitHub Actions Marketplace to authenticate with AWS.

YAML
steps:
- uses: aws-actions/configure-aws-credentials@v1
  with:
    role_arn: arn:aws:iam::<AccountID>:role/<DeployRole>
    role_session_name: GitActionsSession
    external_id: ${GITHUB_RUN_ID}
Step 3: Configure the IAM Role Trust Policy

To ensure that only branches or forks with specific conditions can assume this role, we need to update the trust policy of the IAM role. Specifically, we'll add conditions to the trust policy based on the branch context key passed by GitHub Actions.

JSON
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "",
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::<AccountID>:oidc-provider/<ProviderName>"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "token.actions.githubusercontent.com/branch": ["main", "staging"]
                }
            }
        }
    ]
}
Step 4: Test the Workflow

Finally, we test our updated GitHub Actions workflow to ensure it works as expected. We should see that only pull requests targeting main or staging branches can successfully deploy.

#### Closing Summary
and Common Mistakes/Follow-Ups

In this transition from long-lived keys to OIDC federation, we've ensured better security by leveraging temporary credentials for our deployments. The key elements include setting up an OIDC provider in AWS IAM, configuring the GitHub Actions workflow to authenticate with this provider, updating the trust policy of the IAM role to restrict access based on branch conditions, and thoroughly testing the setup.

#### Common mistakes
might include:

Forgetting to update the trust policy conditions.
Misconfiguring the OIDC provider or client in AWS.
Not properly setting up the context keys required by GitHub Actions for authentication.

#### Follow-ups
could involve discussing additional security measures such as MFA, rotating credentials, and monitoring access logs.

