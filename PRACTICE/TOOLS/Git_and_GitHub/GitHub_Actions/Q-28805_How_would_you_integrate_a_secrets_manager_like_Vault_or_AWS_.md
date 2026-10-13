---
type: "practice"
difficulty: "Expert"
---

# How would you integrate a secrets manager like Vault or AWS Secrets Manager so short-lived secrets are fetched at runtime instead of stored as repo secrets?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Secrets in Actions](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Secrets_in_Actions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To integrate a secrets manager like HashiCorp Vault or AWS Secrets Manager with GitHub Actions, we need to ensure that sensitive information is not hard-coded into our repository and instead fetched at runtime from a secure secrets management system. This approach enhances security by minimizing the exposure of sensitive data in version control systems.

#### Triage

The question requires setting up an environment where secrets are securely managed outside the codebase, ensuring they are only accessible when needed during the execution of GitHub Actions workflows. The main goal is to avoid storing sensitive information directly in repository secrets due to potential security risks and compliance issues.

Step-by-Step Solution
Set Up Secrets Manager:
Configure HashiCorp Vault or AWS Secrets Manager to store necessary secrets such as API keys, database credentials, etc.
Configure GitHub Actions Workflow:
Use the appropriate provider's SDK to fetch these secrets during runtime in your GitHub Actions workflows.

#### Implementation
Steps

Store Secrets in a Secure Repository:

YAML
# Example Vault Configuration
[generic]
path = "secrets"
kv-version = 2
Store sensitive data in the appropriate paths within HashiCorp Vault or AWS Secrets Manager.

GitHub Actions Workflow File (.github/workflows/example.yml):

YAML
name: Fetch Secrets and Run Job

on:
  push:
    branches: [ main ]
  schedule:
    - cron: '0 3 * * *'

jobs:
  fetch-secrets-and-run-job:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Login to HashiCorp Vault
        id: login
        env:
          VAULT_ADDR: ${{ secrets.VAULT_ADDR }}
          VAULT_TOKEN: ${{ secrets.VAULT_TOKEN }}
        run: |
          if [ ! -f ~/.vault-token ]; then
            echo "Token not found, generating..."
            vault login -address=${VAULT_ADDR} ${VAULT_TOKEN}
          fi

      - name: Fetch Secrets
        id: fetch-secrets
        env:
          VAULT_ADDR: ${{ secrets.VAULT_ADDR }}
          VAULT_TOKEN: ${{ secrets.VAULT_TOKEN }}
        run: |
          export S3_ACCESS_KEY=$(vault read -field=access_key ${VAULT_ADDR}/secrets/aws)
          export S3_SECRET_KEY=$(vault read -field=secret_key ${VAULT_ADDR}/secrets/aws)

      - name: Run Job with Secrets
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        run: |
          # Use the fetched secrets in your application or script here

Environment Variables in GitHub Actions:

Define environment variables for sensitive data in the .env file or directly in the workflow, which will be replaced with actual values from Vault during runtime.

#### Closing Summary
and Common Mistakes/Follow-Ups
Ensure that your secrets are stored securely and that access is controlled via roles and permissions.
Use environment-specific configurations to avoid hard-coding secrets across different environments (dev, staging, prod).
Regularly review and update secret management practices as part of your security posture.

#### Example
Follow-Up Questions
How would you handle long-lived secrets like API keys?
What are the best practices for integrating a secrets manager with CI/CD pipelines in other systems besides GitHub Actions?

**Code:**

```yaml
# Example GitHub Actions Workflow
name: Fetch Secrets and Run Job

on:
  push:
    branches: [ main ]
  schedule:
    - cron: '0 3 * * *'

jobs:
  fetch-secrets-and-run-job:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Login to HashiCorp Vault
        id: login
        env:
          VAULT_ADDR: ${{ secrets.VAULT_ADDR }}
          VAULT_TOKEN: ${{ secrets.VAULT_TOKEN }}
        run: |
          if [ ! -f ~/.vault-token ]; then
            echo "Token not found, generating..."
            vault login -address=${VAULT_ADDR} ${VAULT_TOKEN}
          fi

      - name: Fetch Secrets
        id: fetch-secrets
        env:
          VAULT_ADDR: ${{ secrets.VAULT_ADDR }}
          VAULT_TOKEN: ${{ secrets.VAULT_TOKEN }}
        run: |
          export S3_ACCESS_KEY=$(vault read -field=access_key ${VAULT_ADDR}/secrets/aws)
          export S3_SECRET_KEY=$(vault read -field=secret_key ${VAULT_ADDR}/secrets/aws)

      - name: Run Job with Secrets
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        run: |
          # Use the fetched secrets in your application or script here
```

**Flow:**

1. GitHub Actions Workflow
2. Login to Vault
3. Fetch Secrets
4. Run Job with Secrets

