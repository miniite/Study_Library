---
type: "practice"
difficulty: "Basic"
---

# What is the risk of hard-coding a Datadog API key directly inside a pipeline YAML file in the repo?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Secrets in Pipelines](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Secrets_in_Pipelines/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Hard-coding a Datadog API key directly into a CI/CD pipeline YAML file poses significant security risks, as it exposes your credentials to potential threats such as code leaks and unauthorized access. This practice is generally frowned upon in secure development environments.

To understand the full implications, let's break down what happens when you hard-code an API key:

Visibility: The API key becomes visible in source control repositories, making it easy for anyone with repository access to discover sensitive information.
Exposure: CI/CD pipelines often run on shared infrastructure where multiple projects may be deployed or accessed. Hard-coded keys can be extracted and misused by malicious actors.
Auditing Challenges: It becomes difficult to track who has access to the API key, making it harder to audit and manage permissions.

Now, let's explore a step-by-step solution from first principles:

Step 1: Understand the Problem

The core issue is that hard-coding credentials in configuration files can lead to security breaches. We need to ensure that sensitive information like API keys are managed securely.

Step 2: Secure Credential Management

One effective way to manage secrets such as API keys is through a dedicated secret management service, which ensures that sensitive data is not exposed directly within the codebase or pipeline configurations.

#### Example
Workflow:
Secret Storage: Store your Datadog API key in a secure vault managed by tools like HashiCorp Vault, AWS Secrets Manager, or Azure Key Vault.
Environment Variables: Use environment variables to reference these secrets. For example, you can set DATADOG_API_KEY as an environment variable and use it in your CI/CD pipeline configuration.
CI/CD Pipeline Configuration:
YAML
steps:
  - name: Deploy Application
    image: my-app-image
    env:
      DATADOG_API_KEY: ${DATADOG_API_KEY}
Step 3: Environment-Specific Configurations

Ensure that your CI/CD pipeline is configured to use environment-specific variables. This approach minimizes the risk of accidentally leaking sensitive information.

Step 4: Secure Secrets in Version Control

Never commit secrets directly to version control systems like Git. Use pre-commit hooks or other tools to prevent this from happening.

#### Closing Summary
and Common Mistakes/Follow-Ups:

Hard-coding API keys is a poor security practice that can lead to significant risks, such as data breaches and unauthorized access. By implementing secure secret management practices, you can mitigate these risks and ensure your CI/CD pipeline remains robust and compliant with security standards.

#### Common mistakes
include committing sensitive information directly to repositories or failing to use environment variables in CI/CD pipelines. Always follow best practices for managing secrets securely.

**Code:**

```yaml
steps:
  - name: Deploy Application
    image: my-app-image
    env:
      DATADOG_API_KEY: ${DATADOG_API_KEY}
```

**Flow:**

1. Store Secrets in Vault
2. Use Environment Variables
3. CI/CD Pipeline Configurations
4. Deploy Application Safely

