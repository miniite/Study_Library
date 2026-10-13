---
type: "practice"
difficulty: "Expert"
---

# How do you wire GitLab CI to pull secrets from HashiCorp Vault at job runtime and rotate them safely?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Protected Environments](<../../../../TOOLS/GitLab/GitLab_CI/Protected_Environments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To securely manage secrets in a GitLab CI/CD pipeline, we need to integrate HashiCorp Vault as a source of truth for our sensitive data. The goal is to pull these secrets at runtime and use them within our jobs without exposing them directly in the repository or build environment. This process involves several steps: configuring Vault integration, setting up environmental variables, and ensuring secure rotation.

The core idea here is to leverage HashiCorp Vault's strong security features—such as role-based access control (RBAC), encryption at rest, and token management—to safely manage secrets during CI/CD executions.

**Comparison:**

```
Step	Action
1	Configure Vault Integration and Token Management.
2	Define a custom function vault_read to fetch secrets using the token.
3	Use the fetched secret in the CI job script.
4	Ensure secure rotation of Vault tokens and secrets through proper governance policies.
```

**Code:**

```yaml
# .gitlab-ci.yml snippet

stages:
  - deploy

deploy-job:
  stage: deploy
  environment:
    name: production
    url: https://your-production-app-url.com
  script:
    - export PASSWORD=$(vault_read "secret/password")
    - echo "Using password: $PASSWORD"
  before_script:
    # Load Vault token from GitLab CI variable
    - vault_token=$VAULT_TOKEN
  services:
    - docker:dind

# Vault reads function
vault_read() {
  echo $(curl --header "X-Vault-Token: $VAULT_TOKEN" \
              --request GET \
              "http://vault-server/v1/secret/data/${1}" | jq -r '.data.data | to_entries[] | "\(.key)=\(.value)"' | tr '\n' ' ')
}
```

**Example input / output:**

```

Given the following input in .env or directly within the job script:

VAULT_TOKEN=your-vault-token

The output will be:

SHELL
Copy
Using password: your-password-value
```

**Data:**

```json
{
  "VAULT_TOKEN": "your-vault-token",
  "VULTR_VPS_ID": "0123456789abcdef"
}
```

**Flow:**

1. Valid Token
2. Invalid Token
3. CI Job Starts
4. Fetch Secret
5. Vault Token Check
6. RW Access to Vault
7. Secret Fetch
8. Set Environment Variable
9. Job Continues with Secrets
10. Error: Unauthorized

