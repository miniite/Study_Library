---
type: "practice"
difficulty: "Deep"
---

# How do you keep a shared module's schema backward compatible when you must rename an input variable that dozens of stacks already use?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Module Compatibility](<../../../../TOOLS/Terraform/Terraform_Core/Module_Compatibility.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with Terraform, maintaining backward compatibility in a shared module while needing to rename an input variable is crucial for avoiding disruptions across multiple stacks. The key idea here involves leveraging Terraform State Management effectively and using structured state files. By doing so, we can ensure that existing configurations are not broken during the transition.

Terraform's State Management allows you to store infrastructure as code (IaC) in a consistent format, ensuring that changes to modules do not disrupt already provisioned resources. To achieve this, we will follow these steps:

Understand the Impact of Renaming: First, understand what renaming an input variable means and how it affects existing configurations.
Transition Plan: Develop a plan for transitioning from the old variable name to the new one without breaking any stacks that depend on the module.
Use Structured State Files: Ensure that your Terraform state is properly structured to support multiple versions of the same resource.

Let's dive into each step in detail:

Step 1: Understanding the Impact

Renaming an input variable can break existing configurations because the old and new names are not interchangeable. To avoid this, we need to introduce a transitional phase where both the old and new variables are recognized by Terraform.

Step 2: Transition Plan

To handle this transition smoothly:

Introduce a New Variable: Add the new variable with a default value that matches the old one.
Conditionally Use Variables: Implement logic within your module to conditionally use either the old or new variable based on some identifier (e.g., version).
Update Existing Configurations: Modify existing configurations to include both variables, ensuring they are set correctly.
Step 3: Using Structured State Files

To manage state effectively, ensure that:

State Versioning: Use Terraform's state locking and versioning features.
State Backups: Regularly back up your state files to prevent data loss.
Transition Scripts: Write scripts or use tools like terraform import to migrate existing resources from the old variable names to the new ones.

#### Example
Implementation

Let's consider a simple example where we need to rename an input variable called old_var to new_var.

Before Renaming:
HCL
module "example" {
  source = "./shared_module"

  old_var = "value1"
}
After Renaming:

Add a new variable and conditionally use it:

HCL
module "example" {
  source = "./shared_module"

  old_var = var.old_var ?? var.new_var

  # Optionally, set default value for new_var
  new_var = "value2"
}

In the module itself, you can implement logic to handle both variables:

HCL
variable "old_var" {}
variable "new_var" {}

locals {
  var_name = try(local.var_name, var.old_var)
}

resource "aws_instance" "example" {
  # Use either old_var or new_var based on local.var_name
}

#### Closing Summary
and Common Mistakes
Common Mistake: Not planning for the transition carefully can lead to breaking existing configurations.
Best Practice: Always test transitions in a staging environment before applying them to production.

By following these steps, you ensure that your shared Terraform module remains backward compatible while allowing necessary changes to be made.

**Flow:**

1. Define Transition Plan
2. Add New Variable
3. Introduce Conditional Logic
4. Test in Staging Environment
5. Apply to Production

