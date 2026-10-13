---
type: "practice"
difficulty: "Intermediate"
---

# What is an ARM template and how does it enable Infrastructure as Code on Azure?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Azure DevOps and Artifact Governance](<../../../../TOOLS/Azure/Azure_Platform/Azure_DevOps_and_Artifact_Governance/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

ARM (Azure Resource Manager) templates are a powerful way to define, deploy, and manage infrastructure in Azure using JSON or Bicep files. They enable Infrastructure as Code (IaC), allowing developers and DevOps engineers like yourself to write code that defines the desired state of your cloud resources. This approach contrasts with manually configuring resources through the Azure portal, which can be error-prone and less scalable.

At a basic level, ARM templates are JSON or Bicep files that define the resources you need in Azure, such as virtual machines, storage accounts, networks, and more. These templates use resource parameters to specify values like names, locations, and settings, making it easier to manage multiple environments (e.g., development, staging, production) with consistent configurations.

ARM templates work by describing your infrastructure's structure using a declarative approach. This means you define the desired state of your resources—what they should be—and ARM takes care of creating or updating them accordingly. Let’s break this down step-by-step:

Define Resources: You start by defining the resources you need in your template, such as VMs, storage accounts, and networks.
Parameterize Inputs: To make templates flexible across environments, you can parameterize inputs like resource names, locations, and other configurable settings.
Specify Dependencies: ARM templates support specifying dependencies between resources to ensure they are created or updated in the correct order.
Deployment Execution: You use Azure CLI commands (like az deployment group create), PowerShell cmdlets, or the Azure portal to deploy your ARM template.

Here’s a simple example of an ARM template defining a virtual machine:

JSON
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "type": "string",
      "defaultValue": "myVM"
    },
    "adminUsername": {
      "type": "string",
      "defaultValue": "azureuser"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Compute/virtualMachines",
      "name": "[parameters('vmName')]",
      "apiVersion": "2019-07-01",
      "location": "[resourceGroup().location]",
      "properties": {
        "osProfile": {
          "computerName": "[parameters('vmName')]",
          "adminUsername": "[parameters('adminUsername')]"
        },
        // Additional properties...
      }
    }
  ]
}

This template defines a virtual machine with some default parameters. The resources section describes the VM, and you can customize it further by adding more details.

#### Closing Summary
ARM templates are essential for managing infrastructure on Azure using IaC. By defining resources declaratively, parameterizing inputs, specifying dependencies, and deploying them through scripts or tools, you can automate infrastructure management processes, ensuring consistency and reducing errors in cloud deployments.

#### Common Mistakes/Follow-Ups

Not Using Parameters: Always use parameters to make templates flexible across different environments.
Overlooking Dependencies: Ensure that resources are defined with correct dependencies to avoid deployment issues.
Manual Updates: Be cautious about manually updating resources; ARM templates provide a safer and more controlled way to manage changes.

