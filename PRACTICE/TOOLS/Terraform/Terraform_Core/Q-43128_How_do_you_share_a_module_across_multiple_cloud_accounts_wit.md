---
type: "practice"
difficulty: "Deep"
---

# How do you share a module across multiple cloud accounts with different provider credentials while keeping each account's state isolated?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Module Compatibility](<../../../../TOOLS/Terraform/Terraform_Core/Module_Compatibility.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To share a Terraform module across multiple cloud accounts with different provider credentials while maintaining isolation of states, we need to leverage Terraform’s state management and its modular capabilities effectively. This setup ensures that each account has an isolated state but can still use the same reusable modules.

#### Triaging the Question

The core issue is about implementing a multi-account architecture in Terraform where different accounts have distinct provider credentials yet share common modules. We need to ensure isolation of states, which means no accidental cross-talk between different accounts, while reusing the same codebase for consistency and efficiency.

We will make assumptions such as:

Each cloud account has its own unique set of provider credentials.
The shared module is responsible for resource creation and configuration without needing to store state outside Terraform.
We aim to avoid merging or consolidating states across accounts, which could introduce risks like accidental changes in one account affecting another.
Step-by-Step Solution

Define the Module: Create a reusable Terraform module that is agnostic of specific provider credentials and configurations. This module will contain resource definitions but not state management logic.

Provider Configuration in Modules: Use local-exec or null-resource to inject the correct provider configuration during module instantiation. This can be done via variables passed from the parent scope.

Account-Specific States: Ensure each account has its own isolated Terraform state file, typically stored in a version-controlled S3 bucket or another secure storage system.

Sharing Logic Across Accounts: Use Terraform workspaces to manage different environments (e.g., dev, prod) within the same codebase but with distinct states.

Detailed Implementation

Define the Shared Module:

TERRAFORM
# my-shared-module/main.tf
resource "aws_instance" "example" {
  ami           = var.ami_id
  instance_type = var.instance_type
}

Provider Configuration Injection: In your main Terraform file for each account, you can configure the provider and pass variables to the module.

TERRAFORM
# my-main.tf (for dev account)
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
  }
}

provider "aws" {
  region = var.region
  access_key = var.access_key
  secret_key = var.secret_key
}

module "example_instance" {
  source = "./my-shared-module"

  ami_id           = "ami-12345678"
  instance_type    = "t2.micro"
  region           = "us-west-2"
  access_key       = var.access_key_dev
  secret_key       = var.secret_key_dev
}

State Management: For each account, use Terraform workspaces to maintain isolated state.

BASH
# Command for dev workspace
terraform init -work=dev

# Apply the configuration in the dev workspace
terraform apply -workspace=dev

Version Control and Security: Store Terraform files, including states, in a version-controlled repository with proper access controls. Use tools like AWS S3 to manage state files securely.

#### Closing Summary
+ Common Mistakes

In summary, sharing modules across multiple accounts involves careful configuration of provider credentials within the module and leveraging workspaces for isolated states. The key is to ensure that each account’s state remains secure and isolated, while the shared codebase allows for consistent and efficient deployment practices.

#### Common mistakes
include:

Not properly securing and versioning Terraform files and state data.
Failing to use workspaces effectively, leading to unintended state merging or corruption.
Overlooking the necessity of injecting provider-specific configurations into modules via variables.
Additional Considerations
Automation: Consider automating the process using scripts or CI/CD pipelines to handle provider credentials securely.
Security Best Practices: Always follow security best practices when storing and managing Terraform states, especially in multi-account scenarios.

