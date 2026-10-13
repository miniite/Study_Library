---
type: "practice"
difficulty: "Intermediate"
---

# What is Sealed Secrets and how does it help GitOps?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Secrets in GitOps](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Secrets_in_GitOps.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Sealed Secrets is a powerful tool that enhances GitOps practices by ensuring secrets are safely stored in your Git repository while maintaining their encrypted state. In this response, I'll explain what Sealed Secrets is, how it integrates with GitOps, and the key benefits it provides.

#### Triage

The question asks about Sealed Secrets within the context of GitOps. It's important to clarify that Sealed Secrets are part of the Flux CD toolset but can be used in other GitOps environments as well. The core challenge is understanding how secrets management impacts GitOps and why using Sealed Secrets addresses common issues.

#### First Principles Solution

Firstly, let’s understand what a secret is in this context. Secrets include sensitive data such as API keys, database credentials, or encryption keys that need to be stored securely but accessed during deployment. In a GitOps setup, where infrastructure and configuration are version-controlled, storing secrets directly in the repository poses significant security risks.

What is Sealed Secrets?

Sealed Secrets encrypts these secrets using Kubernetes secrets or another encrypted storage mechanism like Vault. This ensures that sensitive information remains protected until it reaches the target environment, typically through a cluster administrator’s decryption process.

How Does It Work with GitOps?
Secret Storage: Developers store their secrets in plain text within a sealed-secrets.yaml file.
Encryption: A tool like sealed-secrets encrypts these plaintext secrets using a provided encryption key or secret from Kubernetes Secrets, AWS KMS, etc.
Decryption at Deployment Time: During deployment, the encrypted secrets are decrypted by the cluster administrator and applied to the correct Kubernetes Secret.
Why Use Sealed Secrets?
Security: Keeps sensitive data out of version-controlled repositories.
Compliance: Aligns with security best practices for storing credentials.
Flexibility: Can be integrated with various key management solutions, enhancing scalability and flexibility.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Sealed Secrets is a critical component in GitOps environments to manage secrets securely while maintaining the benefits of version control. The primary mistake developers often make is storing plain-text secrets directly within their repositories, which can lead to significant security vulnerabilities. Understanding how Sealed Secrets integrates with other tools and workflows ensures robust secret management practices.

Common follow-ups might include discussions about key rotation strategies, integration with other CI/CD pipelines, or specific use cases in different organizational environments.

**Comparison:**

```
Step	Action	Result
1	Store plaintext secret	sealed-secrets.yaml
2	Encrypt secret with kubeseal	Encrypted secret stored in Kubernetes Secret
3	Apply decrypted secret during deployment	Secret applied to the correct namespace and service account
```

**Code:**

```yaml
# Example sealed-secrets.yaml
apiVersion: secretsstore.cattle.io/v1
kind: SecretStore
metadata:
  name: example-secret-store
spec:
  providerRef:
    apiGroup: cattle-toolbox.cattle.dev
    kind: SealedSecretProvider
    name: default

---
apiVersion: v1
kind: Secret
metadata:
  name: example-secrets
data:
  secret-key: Zm9vYmFy
```

**Example input / output:**

```text
Input:
- Developer stores plaintext secrets in sealed-secrets.yaml file.

Output:
- Cluster administrator decrypts and applies the secrets using kubeseal or other tools.
```

**Data:**

```json
{
  "providerRef": {
    "apiGroup": "cattle-toolbox.cattle.dev",
    "kind": "SealedSecretProvider",
    "name": "default"
  }
}
```

**Flow:**

1. Developer Storing Secrets
2. plaintext secrets
3. SealedSecrets tool encryptssecrets
4. Kubernetes Secret
5. Cluster AdministratorDecryption
6. Deployed Application withSecure Secrets

