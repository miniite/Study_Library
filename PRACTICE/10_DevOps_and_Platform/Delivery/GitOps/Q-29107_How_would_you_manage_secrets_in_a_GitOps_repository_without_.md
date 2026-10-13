---
type: "practice"
difficulty: "Intermediate"
---

# How would you manage secrets in a GitOps repository without committing them in plaintext?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Secrets in GitOps](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Secrets_in_GitOps.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When managing secrets in a GitOps repository, the primary challenge is to ensure that sensitive information like API keys or passwords are not committed in plaintext while still being accessible and version-controlled. A common approach is to use encrypted files or secret management tools integrated with your CI/CD pipeline.

First, let's outline the problem: we need to securely store secrets but also make sure they can be referenced by our applications. This involves creating a secure environment for handling secrets during both development and deployment phases.

#### Triaging the Question

The question is probing how you would handle sensitive information in a GitOps setup without exposing it directly in the repository. The key aspects include:

Security: Ensuring that secrets are not committed to the public.
Accessibility: Making sure your applications can access these secrets when needed.
Automation: Integrating this with CI/CD pipelines for seamless deployments.

#### First-Principles Solution

To manage secrets securely, we will leverage a combination of GitOps practices and external secret management tools. Specifically:

Encrypted Files in Repository: Store the secrets as encrypted files or use an environment-specific configuration file.
Secrets Management Tool: Integrate with a secrets management tool like HashiCorp Vault or AWS Secrets Manager to handle encryption, decryption, and access control.
Step-by-Step Solution

Encrypting Secrets:

Use tools like gpg for encrypting files containing sensitive data.
Store the encrypted files in your GitOps repository with a clear indication that they are encrypted and should not be decrypted directly.

Using Environmentspecific Configuration:

Create environment-specific configuration files (e.g., .env or .yml) where secrets can be referenced indirectly rather than stored plaintext.
For example, you might store an API key as a placeholder in the config file: API_KEY=REPLACE_ME.

Secrets Management Tool:

Integrate with a secrets management tool like HashiCorp Vault or AWS Secrets Manager.
Use the secrets tool to manage and retrieve encrypted secrets at runtime.

CI/CD Pipeline Integration:

Configure your CI/CD pipeline to automatically decrypt and inject secrets into your application’s environment variables during deployment.
Ensure that only authorized users have access to the secrets management tool, and follow best practices for managing permissions.

Monitoring and Auditing:

Set up monitoring and logging to track when and how secrets are accessed.
Regularly audit access logs to ensure compliance with security policies.

#### Closing Summary

By using a combination of encrypted files, environment-specific configuration, external secret management tools, and CI/CD pipeline integration, you can securely manage secrets in your GitOps repository. This approach ensures that sensitive information remains protected while still being accessible for development and deployment needs.

#### Common Mistakes/Follow-ups
Not Using Encrypted Files: Directly committing plaintext secrets to the repository.
Lack of Environment-Specific Configurations: Storing identical configuration across environments, leading to security vulnerabilities.
Ignoring Access Control: Failing to restrict access to secret management tools and CI/CD pipeline configurations.

**Comparison:**

```
Step	Action
1	Encrypt secrets
2	Store in repository
3	Use environment-specific config files
4	Integrate with secret management tool
5	Securely inject into deployment pipeline
```

**Code:**

```bash
# Example encrypted file using gpg
gpg --symmetric --output secrets.enc --batch --yes --passphrase "your-passphrase" secrets.txt

# Decryption command in a script (example)
gpg --batch --yes --decrypt --output decrypted-secrets.json secrets.enc
```

**Example input / output:**

```text
# Example CI/CD script snippet for decryption and injection
if [ -f secrets.enc ]; then
  gpg --batch --yes --decrypt --output decrypted-secrets.json secrets.enc
  export API_KEY=$(grep API_KEY decrypted-secrets.json | cut -d '=' -f2)
  export DATABASE_URL=$(grep DATABASE_URL decrypted-secrets.json | cut -d '=' -f2)
fi
```

**Data:**

```yaml
# Example of an environment-specific configuration file (.env)
API_KEY=REPLACE_ME
DATABASE_URL=REPLACE_ME
```

**Flow:**

1. Repository
2. CI/CD Pipeline
3. Secrets Management Tool
4. Decryption and Injection
5. Application

