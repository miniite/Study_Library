---
type: "practice"
difficulty: "Intermediate"
---

# How does Terraform work with Azure, and what is the AzureRM provider?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Azure Fundamentals](<../../../../TOOLS/Azure/Azure_Platform/Azure_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Terraform is an infrastructure as code tool that allows you to define and manage cloud resources in a declarative way. When it comes to working with Azure, Terraform uses providers, which are pre-configured packages that enable Terraform to interact with specific platforms like AWS, Google Cloud, or Azure.

The AzureRM provider (now known as the azurerm provider) is one such package designed specifically for managing Azure resources. It provides a set of resources and operations you can use within your Terraform configuration files to create, update, and delete resources in Azure.

Terraform works with the AzureRM provider by establishing a connection to Azure through an API or RESTful service using authentication credentials. It then uses this connection to read, write, and manage resources according to the definitions provided in your Terraform configuration.

#### Triage
What is Being Probed

The interviewer is probing whether you understand:

How Terraform interacts with external services.
The role of providers like AzureRM.
Authentication mechanisms for connecting to Azure.
Basic operations that can be performed using Terraform and AzureRM, such as creating resources, applying changes, and handling state management.
Step-by-Step Solution

Provider Initialization:

Terraform provider configurations are defined in the provider block within your configuration file. For Azure, this looks like:
HCL
provider "azurerm" {
  features {}
  subscription_id = "your-subscription-id"
  client_id       = "your-client-id"
  client_secret   = "your-client-secret"
  tenant_id       = "your-tenant-id"
}
The provider block sets up the connection to Azure and specifies required credentials.

Resource Definitions:

Resource definitions are created using Terraform's resource syntax, such as:
HCL
resource "azurerm_resource_group" "example" {
  name     = "terraform-resources"
  location = "East US"
}
This example creates an Azure Resource Group named terraform-resources in the East US region.

State Management:

Terraform maintains a state file that tracks all resources created by your configuration. This state is used to apply, refresh, and destroy resources.
You can manage this state using commands like terraform init, terraform plan, terraform apply, and terraform destroy.

Authentication Mechanisms:

AzureRM provider supports various authentication methods such as Managed Service Identity (MSI), Azure Active Directory (AAD) tokens, and service principal credentials.
For example, a service principal can be used with the following configuration:
HCL
provider "azurerm" {
  features {}
  subscription_id = "your-subscription-id"
  client_id       = "your-client-id"
  client_secret   = "your-client-secret"
  tenant_id       = "your-tenant-id"
}

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Common Mistakes

Not initializing Terraform properly before running commands.
Using incorrect or outdated credentials, leading to authentication failures.
Misconfiguring resource names or locations, resulting in errors during resource creation.

#### Follow-Up
Questions:

How do you handle state management and versioning?
What are some best practices for securing your Terraform configuration and credentials?

#### Example
Code Block
HCL
provider "azurerm" {
  features {}
  subscription_id = "your-subscription-id"
  client_id       = "your-client-id"
  client_secret   = "your-client-secret"
  tenant_id       = "your-tenant-id"
}

resource "azurerm_resource_group" "example" {
  name     = "terraform-resources"
  location = "East US"
}

