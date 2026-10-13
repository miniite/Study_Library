---
type: "practice"
difficulty: "Intermediate"
---

# How do you store secrets like API keys and passwords in a CI/CD pipeline?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Secrets in Pipelines](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Secrets_in_Pipelines/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a CI/CD pipeline, securely storing secrets such as API keys and passwords is crucial to maintaining both security and operational efficiency. The challenge lies in balancing secure storage with ease of access during builds and deployments. For an intermediate-level Fullstack SDE-2/3 or Backend SDE-2/3, the key concepts involve understanding how to manage these secrets effectively while following best practices.

The main question here is about securing sensitive information within a CI/CD pipeline. Specifically, it probes knowledge of secure storage mechanisms and best practices in modern software development environments. Common assumptions include that you are familiar with basic CI/CD tools like Jenkins, GitLab CI, or CircleCI, as well as cloud-based services such as AWS Secrets Manager or HashiCorp Vault.

To securely store secrets in a CI/CD pipeline, we can follow these steps:

Identify the Secrets: Determine which credentials and keys need to be stored.
Choose Secure Storage Solutions: Use specialized tools designed for secret management like HashiCorp Vault, AWS Secrets Manager, or Google Cloud Secret Manager.
Environment-Specific Configuration: Store different secrets based on the environment (development, testing, production) in a way that they are accessible only to the required environments.
Secure Access: Ensure that access to these secrets is strictly controlled and logged.
Detailed Solution
Step 1: Identify the Secrets

First, list all sensitive information such as API keys, database passwords, or other authentication tokens that your application needs. This might be part of a configuration file like .env in Node.js applications or application.yml in Spring Boot projects.

Step 2: Choose Secure Storage Solutions

For an intermediate engineer, familiar tools include:

HashiCorp Vault: A secret management tool that can handle various types of secrets and supports multiple storage backends.
AWS Secrets Manager: Provides secure storage for sensitive data using encryption and access control mechanisms.
Google Cloud Secret Manager: Similar to AWS Secrets Manager but tailored for Google Cloud environments.
Step 3: Environment-Specific Configuration

Configure your build pipeline to read these secrets from the chosen secure storage solution. For example, with HashiCorp Vault:

Initialize Vault Client in Code:

JAVASCRIPT

#### cons
t vault = new VaultClient({
    address: "https://vault.example.com",
    token: process.env.VAULT_TOKEN,
    namespace: process.env.VAULT_NAMESPACE || null,
});

Fetch Secrets During Builds:

JAVASCRIPT

#### cons
t secrets = await vault.readSecrets("app/secrets");

Use Secrets in Your Application:

JAVASCRIPT
process.env.API_KEY = secrets.apiKey;
process.env.DATABASE_PASSWORD = secrets.databasePassword;
Step 4: Secure Access

Implement strict access controls to ensure that only authorized users and environments can retrieve these secrets:

Role-Based Access Control (RBAC) in HashiCorp Vault.
IAM Policies in AWS Secrets Manager.
Service Account Credentials for Google Cloud Secret Manager.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In conclusion, securely storing secrets within a CI/CD pipeline involves choosing the right tool, configuring environment-specific settings, and enforcing strict access controls. Common mistakes include not regularly rotating keys or using hard-coded secrets directly in code repositories. Follow-up questions could include discussing key rotation strategies, encryption at rest and transit, and how to handle multi-region deployments.

**Comparison:**

```
Key	Value
API Key	dummy_api_key
Database Password	secret_password
```

**Code:**

```javascript
// Example of fetching secrets from HashiCorp Vault
const vault = new VaultClient({
    address: "https://vault.example.com",
    token: process.env.VAULT_TOKEN,
    namespace: process.env.VAULT_NAMESPACE || null,
});

async function readSecrets() {
    const secrets = await vault.readSecrets("app/secrets");
    return secrets;
}
```

**Example input / output:**

```text
Input:
- `getSecrets()` function call

Output:
- API key and database password read from HashiCorp Vault or equivalent secure storage solution
```

**Data:**

```json
{
  "secrets": {
    "apiKey": "dummy_api_key",
    "databasePassword": "secret_password"
  }
}
```

**Flow:**

1. Start
2. Identify Secrets
3. Choose Secure StorageSolution
4. Environment-SpecificConfiguration
5. Secure Access Controls
6. Fetch and Use Secrets inApplication
7. End

