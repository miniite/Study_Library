---
type: "practice"
difficulty: "Intermediate"
---

# Your CI pipeline needs a short-lived secret to deploy. Walk me through using Vault Agent vs the AppRole auth method for a GitHub Actions runner.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Vault Agent and Auto-auth](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Vault_Agent_and_Auto-auth.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To manage secrets in your CI pipeline, you have two main choices with HashiCorp Vault: using Vault Agent or AppRole authentication. The key differences lie in how they handle secret retrieval and management within a GitHub Actions runner environment.

TL;DR:
Vault Agent: More convenient for short-lived secrets due to auto-renewal capabilities.
AppRole Auth: Requires manual token acquisition, more secure but involves additional steps.

#### Triaging the Question

The question is asking you to compare two methods of authenticating with HashiCorp Vault in a GitHub Actions runner context. The core issue being addressed here is how each method manages short-lived secrets and integrates seamlessly into your CI pipeline.

#### Assumptions
Both scenarios assume that secrets are stored securely within a Vault.
The focus is on short-lived secrets used for deployment purposes.
We're comparing these methods from an operational, security, and ease-of-use perspective in GitHub Actions.

#### First Principles Solution
1. Vault Agent

#### Overview

Vault Agent is a long-running process that connects to Vault and can manage access tokens and pull secrets on demand.
It's particularly useful when you need to periodically fetch short-lived secrets, as it handles token expiration and renewal automatically.

Steps:

Install the Vault Agent in your GitHub Actions runner environment.
Configure the agent with a service-account role that can generate limited-time tokens (e.g., 15 minutes).
In your CI pipeline, use the agent to request secrets as needed.
The agent automatically handles token renewal before expiration.

Advantages:

Simplicity: No need to manually handle token acquisition and renewals.
Auto-Renewal: Ensures that short-lived secrets are always available without manual intervention.
Centralized Management: Easier to manage and revoke access if necessary, as all operations go through the agent.

Disadvantages:

Performance Overhead: Running a separate process can introduce some overhead.
Complexity in Non-DevOps Environments: May require additional setup for environments that are not optimized for running agents (e.g., bare metal servers).
2. AppRole Auth Method

#### Overview

AppRole is an authentication method where a client authenticates with Vault using credentials and receives a token that can be used to access secrets.
It's more suitable when you need fine-grained control over who can access what secrets.

Steps:

Create an AppRole in Vault, which includes generating an ID and Secret Key.
Store the ID and Secret Key securely (e.g., as environment variables).
In your GitHub Actions runner, use these credentials to authenticate with Vault.
Use the generated token to request secrets from Vault.

Advantages:

Fine-Grained Control: Provides more detailed access control over who can read or write secrets.
Security: Since tokens are short-lived and tied to specific actions, they offer better security compared to static credentials.

Disadvantages:

Manual Token Management: Requires manual token acquisition and management, which can be error-prone in CI pipelines.
Complexity: More complex setup and requires additional steps for each action that needs a secret.

#### Closing Summary

When deciding between Vault Agent and AppRole authentication, consider the operational simplicity versus security trade-offs. For a GitHub Actions runner needing short-lived secrets, Vault Agent is often the better choice due to its auto-renewal capabilities and ease of use. However, if you need fine-grained access control or are in an environment where token management complexity is acceptable, AppRole might be more appropriate.

#### Common Mistakes/Follow-Ups
Overlooking Token Renewals: Ensuring that short-lived tokens are properly renewed before expiration.
Security Risks with Agent Setup: Carefully managing the Vault agent's permissions and ensuring it doesn't have broader access than necessary.
AppRole Misconfigurations: Ensuring that AppRole credentials and token management processes are secure and correctly configured.

**Comparison:**

```
Method	Auto-Renewal	Token Management	Security Control	Complexity
Vault Agent	Yes	Automated	Centralized	Low
AppRole	No	Manual	Fine-grained	High
```

**Code:**

```bash
# Example of using Vault Agent in GitHub Actions
steps:
  - name: Install Vault Agent
    run: |
      sudo snap install vault-agent

  - name: Configure Vault Agent
    env:
      VAULT_ADDR: http://vault-service
      VAULT_ROLE_ID: <role-id>
      VAULT_SECRET_ID: <secret-id>
    run: |
      echo 'VAULT_TOKEN=$(vault write auth/approle/login role_id=<role-id> secret_id=<secret-id>)' >> $GITHUB_ENV

  - name: Pull Secret
    env:
      SECRET_NAME: my-deploy-key
    run: |
      vault read --field=${SECRET_NAME} kv/mount/path > ${SECRET_NAME}.key
```

**Example input / output:**

```
Sample GitHub Actions steps for AppRole:
YAML
Copy
steps:
  - name: Configure AppRole Token
    env:
      ROLE_ID: <role-id>
      SECRET_KEY: <secret-key>
    run: |
      export VAULT_TOKEN=$(vault write -time="60s" auth/approle/login role_id=$ROLE_ID secret_id=$SECRET_KEY)

  - name: Pull Secret
    env:
      VAULT_ADDR: http://vault-service
      KUBE_SECRET_NAME: my-deploy-key
    run: |
      vault read -field=${KUBE_SECRET_NAME} kv/mount/path > ${KUBE_SECRET_NAME}.key
```

**Data:**

```
Sample Vault configuration for AppRole:
JSON
Copy
{
  "policies": [
    "deploy-read"
  ],
  "policies_map_output": {
    "deploy-read": ["read"]
  },
  "token_ttl": "30s",
  "token_max_ttl": "60s"
}
```

**Flow:**

1. Yes
2. No
3. Success
4. GitHub Actions
5. Vault Agent
6. Token Renewal?
7. Fetch New Token
8. Use Token to Fetch Secret
9. AppRole
10. Authenticate with AppRole
11. Generate Token
12. Use Token to Fetch Secret

