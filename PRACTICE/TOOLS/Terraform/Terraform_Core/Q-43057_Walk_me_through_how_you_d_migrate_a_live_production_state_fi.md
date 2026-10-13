---
type: "practice"
difficulty: "Deep"
---

# Walk me through how you'd migrate a live production state file from local backend to an S3 remote backend with zero drift and no downtime.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reviewing Plans for State Safety](<../../../../TOOLS/Terraform/Terraform_Core/Reviewing_Plans_for_State_Safety.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To migrate a live production Terraform state file from a local backend to an S3 remote backend with zero drift and no downtime, you need to carefully plan and execute each step thoughtfully. This involves understanding how Terraform's state management works and leveraging modules to ensure consistency across environments.

Firstly, let’s clarify the core problem: migrating the state file without any data loss or drift, ensuring that all resources remain consistent during the transition. The main challenges are managing the state file in a way that it can be seamlessly moved, and ensuring zero downtime during this process.

**Code:**

```hcl
# Example Terraform configuration
locals {
  s3_bucket = "my-production-state-bucket"
}

resource "null_resource" "migrate_state" {
  provisioner "local-exec" {
    command = <<EOT
      # First, fetch the current state file from the local backend
      terraform state pull > state.local.hcl

      # Copy the state to S3 using a custom script or AWS CLI commands
      aws s3 cp state.local.hcl s3://${local.s3_bucket}/terraform.tfstate

      # Optionally, clean up the old state file if no longer needed
      rm state.local.hcl
    EOT
  }
}
```

**Example input / output:**

```

No specific input/output for this step-by-step process is required as it involves local script execution and AWS CLI commands. However, here’s a dry-run example:

SH
Copy
# Example command sequence in the local-exec provisioner
$ terraform state pull > state.local.hcl  # Pulls current state from local backend into file 'state.local.hcl'
$ aws s3 cp state.local.hcl s3://my-production-state-bucket/terraform.tfstate  # Copies the state to S3 bucket
```

**Flow:**

1. Yes
2. No
3. Start
4. Is local backend in use?
5. Fetch current state from localbackend
6. Copy the state file to S3bucket using AWS CLI
7. Cleanup old state file if needed
8. End

