---
type: "practice"
difficulty: "Intermediate"
---

# How do lifecycle meta-arguments like create_before_destroy and prevent_destroy interact with what Terraform records in state?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Refactoring with State Moves](<../../../../TOOLS/Terraform/Terraform_Core/Refactoring_with_State_Moves.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this answer, we will explore how lifecycle meta-arguments such as create_before_destroy and prevent_destroy interact with what Terraform records in its state file when managing resources using Terraform modules.

TL;DR

Terraform uses a state file to record the existence, attributes, and relationships of resources. Lifecycle meta-arguments like create_before_destroy and prevent_destroy influence how these changes are managed within that state. Specifically, create_before_destroy ensures the original resource remains in the state until the new one is successfully created, while prevent_destroy prevents accidental deletion of a resource.

#### Triage

The question asks about the interaction between lifecycle meta-arguments and Terraform's state management. This involves understanding how these arguments affect the state file and the behavior of resources during operations like creation, update, and destruction.

Key Assumptions:
We are using Terraform v0.12 or later.
The state backend is S3 or another durable storage solution.
Resources are being managed within a module.

#### First-Principles Solution

Terraform's lifecycle management is defined by the lifecycle block, which contains several meta-arguments that can be set on individual resources and modules. Two common meta-arguments are create_before_destroy and prevent_destroy.

1. Create Before Destroy (create_before_destroy)

When create_before_destroy is enabled for a resource, it ensures that the original resource remains in the state until the new resource is successfully created. This is particularly useful when you want to avoid leaving resources in an inconsistent state during upgrades or migrations.

#### Example
Configuration:

HCL
resource "aws_instance" "example" {
  lifecycle {
    create_before_destroy = true
  }
}

In this configuration, if aws_instance needs to be replaced due to a version update, Terraform will ensure the original instance remains in the state until the new instance is fully provisioned and functional.

2. Prevent Destroy (prevent_destroy)

The prevent_destroy meta-argument prevents the accidental deletion of a resource by requiring explicit confirmation during the destroy operation. This can be useful for critical resources that should not be deleted without careful consideration.

#### Example
Configuration:

HCL
resource "aws_instance" "example" {
  lifecycle {
    prevent_destroy = true
  }
}

In this case, attempting to terraform destroy the resource will result in an error unless the user explicitly allows it with the -force flag or confirms via a prompt.

Interaction with State Management

State Recording: Both create_before_destroy and prevent_destroy affect how resources are recorded in the state file.

With create_before_destroy, the original resource is kept until the new one is successfully created, ensuring no orphaned resources remain in the state.
prevent_destroy ensures that the resource is not marked for deletion without explicit confirmation.

State Update: During a Terraform run, these settings influence how the state file is updated:

For create_before_destroy, Terraform will update the state to reflect both the original and new resources until the transition is complete.
With prevent_destroy, Terraform will not attempt to remove the resource from the state unless explicitly allowed.

#### Closing Summary

Understanding the interaction between lifecycle meta-arguments and state management is crucial for effective Terraform module design. create_before_destroy ensures a smooth transition of resources, while prevent_destroy enhances security by preventing accidental deletions. Both arguments work in conjunction with the state file to provide robust resource management.

#### Common Mistakes
and Follow-Ups
Misunderstanding Behavior: Not recognizing that create_before_destroy only affects transitions during upgrades.
Overuse of prevent_destroy: Using it for every critical resource, which can lead to unnecessary user prompts.
State Inconsistency: Forgetting the impact on state consistency when transitioning resources.

If you have specific scenarios or need further details on advanced usage, please provide more context!

**Code:**

```hcl
resource "aws_instance" "example" {
  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_instance" "example" {
  lifecycle {
    prevent_destroy = true
  }
}
```

