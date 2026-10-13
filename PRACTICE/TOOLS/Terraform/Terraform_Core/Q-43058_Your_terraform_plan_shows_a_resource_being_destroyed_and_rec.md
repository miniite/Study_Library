---
type: "practice"
difficulty: "Intermediate"
---

# Your terraform plan shows a resource being destroyed and recreated that you only renamed in code. How do you use terraform state mv (or moved blocks) to avoid the destroy?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Refactoring with State Moves](<../../../../TOOLS/Terraform/Terraform_Core/Refactoring_with_State_Moves.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you see a resource being marked as "destroy" and then re-created due to a name change, it's often because Terraform is not recognizing that the resource has been renamed but remains unchanged in its configuration. To avoid this unnecessary destroy/recreate cycle, you can use terraform state mv or moved blocks within your Terraform configuration.

This technique involves marking the resource for migration so that Terraform understands the intended change and updates it without destroying and recreating the resource.

#### Triage
What's Being Asked

The question is probing how well you understand Terraform state management, specifically the terraform state mv command. You need to explain how renaming a resource can be managed without causing unnecessary changes in your infrastructure.

#### First Principles Solution

To avoid the destroy/recreate cycle when renaming resources, use terraform state mv. This command allows you to move an existing resource from one type or name to another within the same Terraform configuration block. Here's a step-by-step approach:

Identify the Resource: Determine which resource is being renamed.
Move the Resource: Use terraform state mv followed by the current and desired names.
Update the Configuration: Ensure that all references to the old name are updated in your configuration.

#### Example

Let's say you have a security group named old-sg which needs to be renamed to new-sg.

Step 1: Identify the Resource

In Terraform, identify the resource by its type and name. For example:

HCL
resource "aws_security_group" "old_sg" {
    ...
}
Step 2: Move the Resource Using terraform state mv

Run the following command in your terminal to rename the security group:

SH
terraform state mv aws_security_group.old_sg aws_security_group.new_sg

This command tells Terraform that you want to move (rename) the resource from old_sg to new_sg.

Step 3: Update References in Configuration and Code

Ensure all references to aws_security_group.old_sg are updated to aws_security_group.new_sg. This includes any provider configurations, module calls, or other dependent resources.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Renaming a resource using terraform state mv is an efficient way to update your infrastructure without causing unnecessary changes. However, it's essential to ensure that all references are updated correctly in the configuration files.

#### Common mistakes
include not running the terraform state mv command or forgetting to update all references in the code. Always verify that both the resource name and its usage are consistent after making this change.

Additional Considerations
State Locking: Ensure no other Terraform processes are using the same state file when renaming resources.
Terraform State File: Sometimes, issues can arise if there's a mismatch between the state file and the configuration. Always validate that the state file is up-to-date before making such changes.

**Code:**

```
SH
Copy
terraform state mv aws_security_group.old_sg aws_security_group.new_sg
```

