---
type: "practice"
difficulty: "Intermediate"
---

# Why is it better to reference secrets from Key Vault in a pipeline instead of hardcoding them in YAML?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Azure Key Vault and Secrets](<../../../../TOOLS/Azure/Azure_Platform/Azure_Key_Vault_and_Secrets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Azure pipelines, referencing secrets from Azure Key Vault rather than hardcoding them is crucial for maintaining security and adhering to best practices. This approach ensures that sensitive information such as API keys, database passwords, or other credentials are not exposed in plain text within the pipeline configuration files.

#### Triage
What the Question Is Asking

The question probes the understanding of secure practices in Azure DevOps pipelines, specifically focusing on the benefits of using Azure Key Vault to manage secrets. The key assumptions here include:

You have a basic knowledge of Azure Key Vault and how it integrates with Azure pipelines.
Understanding that hardcoding sensitive information can lead to security vulnerabilities.

#### First Principles Solution
Step 1: Understand Pipeline Configuration Files

Azure pipelines are defined in YAML files, which are stored in version control systems like Git. These files contain all the necessary steps for building, testing, and deploying applications. Sensitive data such as API keys or passwords should be avoided here to prevent accidental exposure.

Step 2: Introduction of Azure Key Vault

Azure Key Vault is a secure service for storing secrets and cryptographic keys used by applications. It provides a managed environment that ensures the confidentiality, integrity, and availability of these secrets.

Step 3: Fetching Secrets in Pipelines

To use a secret stored in Key Vault within an Azure pipeline:

Create an Access Policy: Grant the appropriate permissions to your pipeline (e.g., Get, List) so it can access the secrets.
Reference the Secret: Use the Azure CLI task or PowerShell task within the YAML file to fetch and use the secret.
Step 4: Benefits of Using Key Vault
Security: Secrets are stored in a secure, managed environment that is isolated from your application codebase. This reduces the risk of accidental exposure.
Scalability: You can manage secrets across multiple environments (e.g., development, staging, production) without modifying pipeline configurations.
Compliance: Key Vault integrates with Azure Security Center to help meet compliance standards.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By referencing secrets from Key Vault in your pipelines, you ensure that sensitive information is securely managed and not exposed within the pipeline YAML files. This approach aligns with best practices for secure application development and deployment. Common mistakes include:

Not setting up proper access policies in Key Vault.
Failing to update pipeline configurations when secrets change or are retired.

#### Follow-ups
could involve discussing additional security measures, such as using Managed Identity for pipelines, and integrating Key Vault with Azure DevOps projects.

**Code:**

```yaml
# Example of referencing a secret from Key Vault in an Azure Pipeline

- task: AzureCLI@2
  inputs:
    azureSubscription: 'YourAzureSubscription'
    scriptType: 'pscore'
    scriptLocation: 'inlineScript'
    inlineScript: |
      $secretValue = az keyvault secret show --name "MySecret" --vault-name "MyKeyVault" --query value -o tsv
      echo "##vso[task.setvariable variable=mySecret;is_secure=true]$secretValue"
  displayName: 'Fetch Secret from Key Vault'
```

