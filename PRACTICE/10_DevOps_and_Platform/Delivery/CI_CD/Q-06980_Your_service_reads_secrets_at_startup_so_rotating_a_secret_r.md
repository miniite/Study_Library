---
type: "practice"
difficulty: "Deep"
---

# Your service reads secrets at startup, so rotating a secret requires a redeploy and rotations became rare; how would you change the pipeline and app so secret rotation is decoupled from deploys?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Secrets in Pipelines](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Secrets_in_Pipelines/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the need for decoupling secret rotations from deploys, we'll revisit our CI/CD pipeline design to ensure that secrets can be rotated independently of application deployments. This requires a more flexible and dynamic approach where secrets are managed separately from the application code.

TL;DR

We will implement a secrets management service that securely stores and rotates secrets. Our CI/CD pipeline will include steps for rotating secrets without needing to redeploy the application, thus decoupling these operations.

#### Triaging the Question

The core of this question is about ensuring that secret rotations can be performed independently of application deployments. The interviewer is probing your understanding of CI/CD best practices, specifically around managing sensitive data like secrets in a secure and scalable manner.

#### Assumptions
and Reasoning from First Principles
Current Setup: Secrets are read at startup and stored directly within the application.
Requirement: Rotate secrets without redeploying the application.

#### Cons
traints: Ensure the solution is secure, maintainable, and fits within a CI/CD workflow.

#### Solution

To achieve this decoupling, we need to introduce a secrets management service that abstracts away direct access to secrets from the application code. This involves modifying both the pipeline and the application.

Step 1: Introduce a Secrets Management Service

We'll use an external secrets management tool like HashiCorp Vault or AWS Secrets Manager. These services provide secure storage, encryption at rest, and dynamic secret rotation capabilities.

Fetch secrets from config
Application
Secrets Management Service
Database/Backend Services
Step 2: Modify the Application

The application will fetch secrets dynamically during startup. The configuration will be updated to reference the secrets management service URL and necessary credentials.

PYTHON
# Example of fetching secrets in Python using a client library for Vault
import hvac

def get_secrets_from_vault():
    client = hvac.Client(url="https://vault.example.com", token="your-vault-token")
    secret_path = "myapp/secrets"
    return client.secrets.kv.v2.read_secret_version(path=secret_path)
Step 3: Update CI/CD Pipeline

The pipeline will be updated to include steps for rotating secrets. This can involve creating and updating secrets in the management service directly.

YAML
pipeline:
  - step: ci
    script:
      - echo "Running CI tests..."
      # Other test steps

  - step: cd
    when: manual
    script:
      - echo "Starting deployment pipeline"
      - mvn deploy         # Example for a Java app using Maven
      - vault kv put myapp/secrets <new_secrets.json>   # Update secrets in Vault
Step 4: Secret Rotation Strategy

We will implement a strategy where secret rotations are managed by the CI/CD pipeline. This could involve automated or manual steps, depending on the risk level and business requirements.

Trigger rotation
CI/CD Pipeline
Secrets Management Service
New Secrets Generated
Notify Applications to FetchNew Secrets

#### Closing Summary

By implementing a secrets management service, updating our application to fetch secrets dynamically, and modifying the CI/CD pipeline to handle secret rotations independently of deployments, we ensure that sensitive data can be managed securely and efficiently. This approach provides a robust solution for managing secrets while maintaining separation between deployment and secret rotation processes.

#### Common Mistakes/Follow-Ups
Mistake 1: Failing to secure the secrets management service properly.
Mistake 2: Not having a clear process for rolling out changes without disrupting services.

#### Follow-Up
Discussing additional security measures like encryption in transit and at rest, and best practices for managing tokens/credentials.

**Comparison:**

```
Step	Description	Tool/Service
1	Introduce a secrets management service	HashiCorp Vault, AWS Secrets Manager
2	Modify application code to fetch secrets dynamically	N/A
3	Update CI/CD pipeline for secret rotations	Jenkins, GitLab CI
4	Implement rotation strategy	Vault, custom scripts
```

**Code:**

```python
# Example of fetching secrets using hvac library
import hvac

def get_secrets_from_vault():
    client = hvac.Client(url="https://vault.example.com", token="your-vault-token")
    secret_path = "myapp/secrets"
    return client.secrets.kv.v2.read_secret_version(path=secret_path)
```

**Example input / output:**

| Input | Action          | Output                       |
|-------|-----------------|------------------------------|
| Manually update secrets in Vault | Notify applications to fetch new secrets | Applications update their configuration with the latest secret values. |

**Data:**

```json
{
  "secrets": {
    "db_password": "new_secure_password",
    "api_key": "updated_api_token"
  }
}
```

**Flow:**

1. CI/CD Trigger
2. Secret Rotation
3. Generate New Secrets
4. Notify Applications

