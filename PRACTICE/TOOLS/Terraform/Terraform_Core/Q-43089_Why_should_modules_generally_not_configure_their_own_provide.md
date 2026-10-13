---
type: "practice"
difficulty: "Deep"
---

# Why should modules generally not configure their own providers or backends? What breaks when a shared module hardcodes a provider block?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Module Compatibility](<../../../../TOOLS/Terraform/Terraform_Core/Module_Compatibility.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When discussing Terraform, it's crucial to understand that each module and its state are intended to be independent and interchangeable pieces of code. However, allowing modules to configure their own providers or backends can lead to several issues, particularly when these configurations are hardcoded. In this answer, we'll delve into why this is generally a bad practice, exploring the potential pitfalls and how they break the modularity and reusability principles that Terraform aims to uphold.

Terraform State & Modules: A Quick Overview

Before diving deep, let's recap what modules and state mean in Terraform. Modules are reusable pieces of code that encapsulate a set of resources with common configurations. They provide a way to abstract complex resource sets into manageable units. The Terraform state file stores the current state of your infrastructure, including details about each resource created by Terraform.

Why Should Modules Generally Not Configure Their Own Providers or Backends?
Modularity and Reusability: One core principle of modules is their ability to be reused across multiple projects without modification. Hardcoding provider configurations within a module violates this principle because it makes the module dependent on specific environments.
Inconsistent Environments: Each project might have different requirements for its infrastructure, such as using AWS, GCP, or both. Hardcoded provider blocks in shared modules can lead to inconsistencies across projects, making migrations and updates cumbersome.
What Breaks When a Shared Module Hardcodes a Provider Block?
Project Independence: If a module hardcodes a provider block, it tightly couples the state of that module with the specific infrastructure provider being used. This means that any project using this module must also use the same provider, which can be problematic if multiple projects need to support different providers.
State Management Issues: The Terraform state is linked to both the configuration and the resources created by a module. If a shared module hardcodes its own provider block, it creates a situation where the state cannot be easily managed independently of other modules. For instance, changing the backend or provider in one project might inadvertently affect another that uses the same module.
Versioning and Migrations: Hardcoding providers within a module can complicate versioning and migration processes. If a project updates its Terraform setup to support a different provider, it must ensure that all modules using this setup are updated as well, which can be error-prone.
Concrete Example

#### Cons
ider a scenario where you have a shared module for managing VPCs (vpc_module). This module is intended to be used across multiple projects. If vpc_module hardcodes an AWS provider block:

HCL
provider "aws" {
  region = "us-east-1"
}

This hardcoded configuration means that any project using vpc_module will always use the AWS provider with a specific region, making it impossible to use GCP or other providers. If another project needs to use GCP and relies on vpc_module, it would have to either duplicate the module code (bad practice) or create a new version of vpc_module, leading to unnecessary complexity.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, modules should avoid configuring their own providers or backends because this hinders modularity and reusability. It can lead to inconsistent environments across projects and complicate state management and versioning processes. Best practices recommend that modules rely on the caller's configuration for provider settings.

#### Common mistakes
include:

Hardcoding provider blocks within shared modules.
Ignoring the environment-specific requirements of each project when creating reusable modules.
Not considering how changes in one module might affect others using it.

#### Follow-up
questions might explore specific scenarios, such as managing multiple environments (dev, staging, prod) or dealing with hybrid cloud setups where different projects require different providers. Understanding these nuances will help ensure that your Terraform configurations are both flexible and maintainable.

**Code:**

```hcl
# Example of a problematic module configuration
provider "aws" {
  region = "us-east-1"
}

resource "aws_vpc" "example" {
  cidr_block = "10.0.0.0/16"
}
```

**Flow:**

1. Yes
2. No
3. Module with hardcodedprovider
4. Is this project using AWS?
5. Conflict with GCP projects
6. Stale state management issues
7. Migrate to versioned modulesupport multiple providers
8. Manual updates required forconsistency

