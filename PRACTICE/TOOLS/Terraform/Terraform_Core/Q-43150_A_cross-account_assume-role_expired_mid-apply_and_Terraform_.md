---
type: "practice"
difficulty: "Deep"
---

# A cross-account assume-role expired mid-apply and Terraform left resources partially created and partly in state. How do you safely resume?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reviewing Plans for State Safety](<../../../../TOOLS/Terraform/Terraform_Core/Reviewing_Plans_for_State_Safety.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address a scenario where a cross-account assume-role expires during an apply, leaving Terraform in a partially applied state, we need to ensure that the state is consistent and avoid any unintended actions or data loss. The solution involves leveraging Terraform State management effectively and using modules for reusability.

First, understand that when Terraform encounters an error mid-apply, it leaves behind a stale state file with incomplete changes. This can lead to inconsistent resources across accounts. We need to safely resume the apply process without losing work or risking further inconsistencies.

#### Triaging the Question

The question is essentially asking how we can manage and resolve a partially applied Terraform configuration when an assume-role expires mid-apply. The interviewer is probing your understanding of:

Terraform State Management: How state files are managed, especially in cross-account scenarios.
Error Handling in Terraform: Best practices for dealing with errors during apply operations.
Reusability and Modularity: The use of modules to simplify resource management across multiple accounts.

#### Assumptions
We have a multi-account environment where resources are created using a cross-account assume-role.
An error occurred mid-apply due to the expiration of the assumed role, leading to an inconsistent state file.
Resources might be partially created and need to be brought into a consistent state.

#### First-Principles Solution

Backup the Current State:

Before making any changes, always back up the current state using terraform show.

Validate the State:

Use terraform apply --auto-approve in a dry-run mode to see what Terraform plans to do.
Identify which resources are partially created and where the process stopped.

Use Modules for Reusability:

Modularize your configuration using Terraform modules to manage resources consistently across different accounts.

Reapply with Fresh Assumptions:

Terminate or refresh the expired state by running terraform destroy --auto-approve if necessary, and then re-run the apply process.

Sync State Across Accounts:

Ensure that any changes are applied to all relevant accounts by managing state files correctly.
Step-by-Step Solution

Backup Current State:

SH
terraform show > current_state.tfstate.json

Dry-Run the Apply Process:

SH
terraform apply --auto-approve -json > dry_run_plan.json

Analyze the Dry-Run Plan:

Inspect dry_run_plan.json to understand which resources are in an inconsistent state.

Identify and Clean Up:

SH
# Example: Remove partial resource states
terraform import <resource_type> <resource_id>
terraform destroy --auto-approve

Reapply with Fresh Assumptions:

Ensure the assumed role is valid before starting.
Apply the configuration again to ensure a consistent state.

#### Closing Summary
and Common Mistakes/Follow-Ups

To safely resume after an assume-role expiration, it’s crucial to follow best practices in state management and error handling. Always backup your current state, validate changes using dry-runs, and use modules for reusability. The key is to ensure that the process starts with a valid assumption role to avoid further inconsistencies.

#### Common mistakes
include:

Overwriting the state file without backing up.
Not validating the apply plan properly.
Failing to modularize configurations, leading to complex and error-prone processes.

#### Follow-ups
might involve deeper discussions on how to automate these steps or integrate them into CI/CD pipelines for better reliability.

**Comparison:**

```
Action	Description
Backup State	terraform show > current_state.tfstate.json
Dry-Run Apply	terraform apply --auto-approve -json > dry_run_plan.json
Import Resources	terraform import <resource_type> <resource_id>
Destroy Partial	terraform destroy --auto-approve
```

**Code:**

```
SH
Copy
terraform show > current_state.tfstate.json

# Dry-run the apply process
terraform apply --auto-approve -json > dry_run_plan.json

# Analyze the dry run plan
cat dry_run_plan.json

# Clean up partial resources if necessary
terraform import <resource_type> <resource_id>
terraform destroy --auto-approve
```

**Example input / output:**

```text
# Example output from a dry run plan
{
  "planned_values": [
    {
      "resource_type": "aws_s3_bucket",
      "changes": {
        "create": {
          "new_value": {
            "bucket": "example-bucket",
            "versioning_configuration": {
              "status": "Enabled"
            }
          },
          "old_value": null
        }
      }
    },
    {
      "resource_type": "aws_ebs_volume",
      "changes": {
        "create": {
          "new_value": {
            "availability_zone": "us-west-2a",
            "size": 10,
            "type": "gp3"
          },
          "old_value": null
        }
      }
    }
  ]
}
```

**Data:**

```json
{
  "resources": [
    {
      "type": "aws_s3_bucket",
      "name": "example-bucket"
    },
    {
      "type": "aws_ebs_volume",
      "name": "example-volume"
    }
  ]
}
```

**Flow:**

1. Backup Current State
2. Show
3. Dry-Run Apply
4. Analyze Dry-Run Plan
5. Identify and Clean Up PartialResources if Necessary
6. Reapply with FreshAssumptions

