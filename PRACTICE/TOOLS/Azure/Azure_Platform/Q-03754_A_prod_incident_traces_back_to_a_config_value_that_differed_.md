---
type: "practice"
difficulty: "Intermediate"
---

# A prod incident traces back to a config value that differed across environments; how do you centralize config with App Configuration and feature flags on Azure?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Azure Pipelines and Release Approvals](<../../../../TOOLS/Azure/Azure_Platform/Azure_DevOps_and_Artifact_Governance/Azure_Pipelines_and_Release_Approvals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of differing configuration values in various environments, we can leverage Azure App Configuration and Feature Flags to achieve centralized management. This approach ensures consistency across all environments and allows us to manage configurations dynamically without redeploying code.

TL;DR:
Centralize Configurations: Use Azure App Configuration for storing and managing application settings.
Feature Flags: Implement feature flags using Azure App Configuration and Azure DevOps pipelines.
Benefits: Centralized management, easier rollbacks, and flexible deployments.

#### Triaging the Question

The question revolves around managing configuration values across different environments (e.g., development, staging, production) to avoid issues like prod incidents. The core of this problem is ensuring that all environments use the same consistent configurations without manually updating each environment. Azure App Configuration and Feature Flags are key tools for achieving this.

Step-by-Step Solution
1. Set Up Azure App Configuration

Azure App Configuration allows you to store, manage, and deliver application settings securely across multiple services. Here’s how to set it up:

BASH
az appconfig create --name <app-config-name> --location <region>

This command creates an Azure App Configuration instance.

2. Store Configurations in App Configuration

You can store key-value pairs representing your application configurations in the Azure portal or using SDKs. For example, you might have settings like API_URL, STORAGE_ACCOUNT_NAME, etc.

BASH
az appconfig kv set --name <app-config-name> --key API_URL --value https://myapi.example.com
3. Retrieve Configurations in Applications

In your application code, you can retrieve these values using the Azure SDKs or by making HTTP requests to App Configuration.

CSHARP
using Microsoft.Azure.AppConfiguration;
using System;

class Program
{
    static void Main(string[] args)
    {
        var builder = new ConfigurationBuilder()
            .AddAzureAppConfiguration(
                connectionString: Environment.GetEnvironmentVariable("AZURE_APPCONFIG_CONNECTION"),
                keyFilter: k => true,
                opts => { /* Additional options */ });

        IConfiguration config = builder.Build();
        string apiUrl = config["API_URL"];

#### Cons
ole.WriteLine($"API URL: {apiUrl}");
    }
}
4. Implement Feature Flags

Feature flags allow you to control the behavior of your application dynamically without redeploying code. You can manage these using Azure App Configuration as well.

Create Features in App Configuration:

BASH
az appconfig kv set --name <app-config-name> --key USE_NEW_FEATURE --value true

Implement Feature Checks in Code:

CSHARP
if (config["USE_NEW_FEATURE"].ToLower() == "true")
{
    // Enable new feature logic
}
else
{
    // Use old implementation
}
5. Use Azure DevOps for Deployment Pipelines

Azure DevOps pipelines can be configured to automatically deploy your application configurations from App Configuration based on environment tags or branches.

YAML
trigger:
- master

pool:
  vmImage: 'ubuntu-latest'

steps:
- task: AzureAppConfiguration@1
  inputs:
    azureSubscription: '<subscription-name>'
    appName: '<app-config-name>'
    slotSettingPrefix: '$(Build.SourceBranchName)'

#### Closing Summary
and Common Mistakes

Centralizing configurations with Azure App Configuration ensures that all environments use the same values, reducing the risk of incidents caused by configuration drift. However, common mistakes include:

Not Using Environment-Specific Configurations: Always ensure you have environment-specific keys to avoid mixing settings.
Hardcoding Values: Avoid hardcoding sensitive information like API keys or credentials directly into your code.
Ignoring Feature Flags: Properly manage and update feature flags as part of your release process.

By following these steps, you can effectively centralize configurations and manage features dynamically in Azure environments.

