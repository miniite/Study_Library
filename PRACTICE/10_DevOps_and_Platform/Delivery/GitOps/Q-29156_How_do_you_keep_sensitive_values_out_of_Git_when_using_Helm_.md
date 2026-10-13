---
type: "practice"
difficulty: "Intermediate"
---

# How do you keep sensitive values out of Git when using Helm values files?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Secrets in GitOps](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Secrets_in_GitOps.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure sensitive values are kept out of Git when using Helm values files, we need to adopt a strategy that separates secret data from application code and configuration. This approach leverages GitOps principles by keeping the configuration in version control while storing secrets securely outside of it. Here’s how you can achieve this:

#### Triage
What's Being Probed

The question is asking about securing sensitive information like database credentials, API keys, or encryption keys used within Helm charts. The key idea here is to understand how to manage these secrets properly without exposing them in your Git repository.

Step-by-Step Solution from First Principles
Identify Sensitive Values: Start by identifying which values are sensitive and need protection.
Use Helm Secrets: Utilize helm-secret or values-scope plugins to handle sensitive data.
Store Secrets Securely: Store these secrets in an external secret management system such as HashiCorp Vault, AWS Secrets Manager, or Kubernetes Secret objects.
Reference Secrets in Values Files: Update your Helm values files to reference the secure storage instead of embedding them directly.
Detailed Steps

Identify Sensitive Values:

List all environment-specific sensitive data used by your application.
Common examples include database passwords, API keys, and AWS credentials.

Use Helm Secrets Plugins:

helm-secret: This plugin allows you to store secrets in a secure manner outside of the Git repository.
values-scope: Another useful tool that helps manage different scopes for your values files.

Store Secrets Securely:

Use HashiCorp Vault or AWS Secrets Manager to store sensitive data securely.
BASH
# Example using HashiCorp Vault
vault kv put secret/myapp password=mypassword api_key=myapikey
For Kubernetes, use the kubernetes.io/dockerconfigjson type for Docker credentials:
YAML
apiVersion: v1
kind: Secret
metadata:
  name: myapp-docker-registry
  namespace: default
data:
  .dockerconfigjson: <base64_encoded_docker_config>

Reference Secrets in Values Files:

Update your Helm values files to reference the secure storage.
YAML
# Example Helm values.yaml using secret references
myapp:
  passwordSecretRef: vault-secret/myapp-password
  apiKeySecretRef: vault-secret/myapp-api-key
Ensure you have a values-scope or equivalent plugin configured in your Helm setup.

#### Closing Summary
and Common Mistakes/Follow-Ups

By separating sensitive values from Git, we ensure that critical data is protected and managed securely. The key steps involve identifying the necessary secrets, using appropriate tools like Helm plugins for secret management, storing these securely, and referencing them appropriately in your configuration files. Some common mistakes include not updating existing configurations to use secure storage or embedding secrets directly into Git repositories.

To further deepen understanding, consider exploring other best practices such as using encrypted values in Git and implementing a robust access control strategy for the secret store.

**Comparison:**

```
Secret Type	Description
Opaque	Basic type for non-standard secrets
Docker	For storing Docker configuration json
```

**Code:**

```yaml
# Example Helm Secret YAML
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secret
type: Opaque
data:
  password: <base64_encoded_password>
  api_key: <base64_encoded_api_key>
```

**Example input / output:**

```text
# Example of a Helm values file using secret references
myapp:
  passwordSecretRef: vault-secret/myapp-password
  apiKeySecretRef: vault-secret/myapp-api-key
```

**Data:**

```json
{
  "password": "cGFzc3VtcGFzcw==",
  "api_key": "bWFpY2thLWtleQ=="
}
```

**Flow:**

1. Identify Sensitive Values
2. Use Helm Secrets Plugin
3. Store Secrets Securely (e.g.,HashiCorp Vault)
4. Reference Secrets in ValuesFiles
5. Ensure Secure Access Control

