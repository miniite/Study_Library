---
type: "practice"
difficulty: "Intermediate"
---

# How would you write an 'az' CLI command to create a resource group and a storage account?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Azure DevOps and Artifact Governance](<../../../../TOOLS/Azure/Azure_Platform/Azure_DevOps_and_Artifact_Governance/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question, we need to understand how to use Azure's az CLI (Command Line Interface) effectively to manage resources. This task involves creating two key Azure resources: a resource group and a storage account.

#### Triaging the Question

The interviewer is likely probing your understanding of Azure command structure, particularly focusing on the az CLI commands for resource management. Key assumptions include:

You are familiar with basic Azure concepts such as resource groups and storage accounts.
You can construct correct az commands using proper syntax and arguments.
Step-by-Step Solution

First-principles reasoning involves understanding that each command in az has a specific structure, where we need to specify the operation (create), the type of resource (resource group or storage account), and any required parameters like names and locations. Here’s how you would do it:

Step 1: Create a Resource Group

To create a resource group in Azure using the az CLI, use the following command:

BASH
az group create --name <RESOURCE_GROUP_NAME> --location <LOCATION>

Here, <RESOURCE_GROUP_NAME> is the name you want to give to your resource group and <LOCATION> specifies where the resource group should be created (e.g., "eastus", "westeurope").

Step 2: Create a Storage Account

To create a storage account, use this command:

BASH
az storage account create --name <STORAGE_ACCOUNT_NAME> --location <LOCATION> --resource-group <RESOURCE_GROUP_NAME>

In this command, <STORAGE_ACCOUNT_NAME> is the name you want to give to your storage account. The --resource-group parameter ensures that the storage account is created within the specified resource group.

Full Example

Let's say you want to create a resource group named "myResourceGroup" and a storage account called "mystorageAccount". Here’s how you would write the commands:

Create the Resource Group:

BASH
az group create --name myResourceGroup --location eastus

Create the Storage Account within the Existing Resource Group:

BASH
az storage account create --name mystorageAccount --location eastus --resource-group myResourceGroup

#### Closing Summary
and Common Mistakes

To summarize, when creating resources in Azure using az CLI commands, always ensure you specify the correct resource type (e.g., resource group or storage account), the location where they should be created, and any necessary parameters like names and groups. Misunderstanding the parameter order or omitting required arguments can lead to errors.

#### Common mistakes
include:

Forgetting to use --resource-group when creating a resource that needs to reside within an existing resource group.
Incorrectly specifying the location name (e.g., using "east usa" instead of "eastus").
Not checking if the resource group already exists, which can result in unnecessary commands or errors.

By following these steps and avoiding common pitfalls, you can efficiently manage your Azure resources through az CLI commands.

**Code:**

```bash
# Example commands to create a resource group and storage account

# Step 1: Create Resource Group
az group create --name myResourceGroup --location eastus

# Step 2: Create Storage Account within the Existing Resource Group
az storage account create --name mystorageAccount --location eastus --resource-group myResourceGroup
```

