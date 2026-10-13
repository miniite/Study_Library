---
type: "practice"
difficulty: "Intermediate"
---

# How do you handle secrets for ephemeral preview environments that spin up per pull request?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Keeping Secrets out of Env and Logs](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Keeping_Secrets_out_of_Env_and_Logs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To manage secrets in ephemeral preview environments triggered by pull requests, we need a robust yet flexible solution to ensure secure handling and usage of sensitive data without compromising security or slowing down development cycles. This involves understanding the environment's nature, identifying key secrets, choosing an appropriate secrets management tool like Hashicorp Vault, and implementing a strategy that dynamically provisions and de-provisions these secrets as needed.

The core idea is to leverage Hashicorp Vault for its robust secret management capabilities. Vault provides granular access controls, encryption at rest, and secure secrets rotation, making it well-suited for managing secrets in transient environments like pull requests. We can use Vault's APIs or integrations with CI/CD tools to automate the fetching of secrets on-demand.

**Code:**

```python
import hvac

# Initialize Vault client with your Vault URL and token
client = hvac.Client(url="https://vault.example.com", token="your-vault-token")

# Function to fetch secrets from Vault based on environment name
def get_secrets(environment):
    secret_path = f"secrets/{environment}"
    try:
        response = client.secrets.kv.v2.read_secret_version(path=secret_path)
        return response["data"]["data"]
    except hvac.exceptions.InvalidPath:
        print(f"No secrets found for {environment}")
        return {}
```

**Example input / output:**

```text
# Example input and output from the get_secrets function
get_secrets("dev-preview-123")
# Output example (assuming the secrets are JSON):
{
    "database_username": "db_user",
    "database_password": "db_pass",
    "api_key": "api_secret"
}
```

**Flow:**

1. CI/CD triggers PR
2. Create preview environment
3. Determine secret requirementsfor the new env
4. Fetch secrets from Vault usingthe get_secrets function
5. Configure environment withfetched secrets
6. Environment becomesavailable and running

