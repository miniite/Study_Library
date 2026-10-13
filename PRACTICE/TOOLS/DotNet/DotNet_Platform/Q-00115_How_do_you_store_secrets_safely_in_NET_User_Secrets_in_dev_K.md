---
type: "practice"
difficulty: "Intermediate"
---

# How do you store secrets safely in .NET (User Secrets in dev, Key Vault or env in prod) instead of appsettings?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Thread Safety and Locking](<../../../../TOOLS/DotNet/DotNet_Platform/Thread_Safety_and_Locking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When storing secrets such as API keys, database connection strings, and other sensitive information in a .NET application, it's crucial to ensure that these secrets are stored securely, especially when transitioning from development to production environments. The common approaches include using User Secrets for development, Azure Key Vault for production, and environment variables (env) as an intermediate or temporary measure.

#### Triage

The question is essentially about how to manage sensitive information in a .NET application across different environments—development, staging, and production. It’s probing your understanding of best practices in secure coding and configuration management, particularly focusing on the use of .NET-specific tools like User Secrets and Azure Key Vault.

#### First-Principles Solution

User Secrets for Development:

Use dotnet user-secrets to store secrets locally.
This is useful during development when you want a quick way to manage sensitive information without committing it to version control.

Environment Variables (Env) in Staging and Production:

Set environment variables using the launchSettings.json file or directly through OS configuration.
These are used for deployment environments where secrets need to be managed externally, ensuring that sensitive data is not committed to source control.

Azure Key Vault in Production:

Store secrets in Azure Key Vault and use Managed Identity or access tokens for secure retrieval within your application.
This ensures that the secrets are stored securely and can be accessed by your application with minimal configuration overhead.
Step-by-Step Implementation

User Secrets Setup (Development):

SH
dotnet user-secrets add --name "YourSecretName" --value "your_secret_value"
Store secrets in the secrets.json file, which is located at .usersecrets/YourProjectName.

Environment Variables Setup:

Modify the launchSettings.json to define environment-specific settings.
JSON
{
  "profiles": {
    "IIS Express": {
      "commandName": "IISExpress",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    "YourProjectName": {
      "commandName": "Project",
      "launchBrowser": true,
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Staging"
      }
    }
  }
}

Azure Key Vault Setup (Production):

Create an Azure Key Vault and store secrets there.
Use Managed Identity to grant access to your application.
CSHARP
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

public void ConfigureServices(IServiceCollection services)
{
    services.AddKeyVaultSecretsManager(options =>
    {
        options.ConfigureFromEnvironmentVariables();
        options.ConfigureFromAppSettings();
    });

    var keyVaultEndpoint = Environment.GetEnvironmentVariable("KEY_VAULT_URL");
    if (!string.IsNullOrEmpty(keyVaultEndpoint))
    {
        services.AddAzureKeyVault(new Uri(keyVaultEndpoint), new DefaultAzureCredential());
    }
}

#### Closing Summary
and Common Mistakes
Mistake 1: Storing secrets in appsettings.json or other version-controlled files.
Mistake 2: Hardcoding sensitive information directly into the source code.
Mistake 3: Not using environment-specific configurations to manage secrets across different deployment stages.

By following these steps, you ensure that your application is secure and compliant with best practices for managing sensitive data in a .NET application.

