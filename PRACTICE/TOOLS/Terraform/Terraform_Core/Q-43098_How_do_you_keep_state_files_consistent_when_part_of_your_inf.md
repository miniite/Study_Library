---
type: "practice"
difficulty: "Deep"
---

# How do you keep state files consistent when part of your infra is in Terraform Cloud and part is in a self-managed S3 backend?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reviewing Plans for State Safety](<../../../../TOOLS/Terraform/Terraform_Core/Reviewing_Plans_for_State_Safety.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When managing infrastructure as code (IaC) with Terraform, ensuring consistency across different backends can be challenging, especially when you have some state files managed by Terraform Cloud and others stored in a self-managed S3 backend. This problem is particularly relevant for teams that are transitioning from self-managed solutions to the more robust offerings of Terraform Cloud. Let’s break down how to manage this scenario step-by-step.

Firstly, the question asks about keeping state files consistent between Terraform Cloud and a self-managed S3 backend. We need to ensure that both environments maintain an accurate and up-to-date representation of our infrastructure. The key challenge here is managing the state file format, versioning, and consistency checks across these different storage solutions.

Terraform Cloud provides built-in support for managing state files in various backends, including remote S3 storage, which can be used to house a self-managed backend. However, since some of your infrastructure might still use the local file system or another backend managed by Terraform CLI, we need to ensure that these different parts of our environment are in sync.

To achieve this consistency:

Define State Backends: Understand and configure both Terraform Cloud and S3 backends properly.

#### Cons
istent State Format: Ensure that the state files adhere to a consistent format across all environments.
Versioning: Implement versioning strategies to track changes in your infrastructure.
Sync Mechanisms: Develop or adopt mechanisms to manually or automatically sync state files between Terraform Cloud and S3.

Now, let’s explore each of these steps in detail:

#### Triaging the Question

The question is essentially about managing consistency across multiple Terraform backends—specifically, ensuring that state files managed by Terraform Cloud and a self-managed S3 backend remain synchronized. The key areas being probed are:

Configuration Management: How you set up and manage different backend configurations.
State File Integrity: Ensuring the integrity of state files across different environments.
Version Control: Implementing strategies to track changes in your infrastructure.
Step-by-Step Solution

Define State Backends:

For Terraform Cloud, configure it as a remote backend using the remote_state module or through CLI commands.
HCL
terraform {
  required_version = ">= 0.12.6"
  backend "remote" {
    organization = "<org-name>"
    workspaces {
      name = "my-workspace"
    }
  }
}
For the self-managed S3 backend, use the s3 backend with appropriate credentials and bucket configuration.
HCL
terraform {
  required_version = ">= 0.12.6"
  backend "s3" {
    region                  = "us-west-2"
    bucket                  = "<bucket-name>"
    key                     = "path/to/state/my-workspace.tfstate"
    dynamodb_table          = "<dynamodb-table>"
    encrypt_key             = false
    encrypt_with            = "<kms-key-id>"
  }
}

#### Cons
istent State Format:

Ensure that the state files in both environments are in the same format (Terraform’s JSON output).
You can use terraform state list and terraform show to inspect and compare the states.

Versioning:

Implement versioning strategies using Terraform Cloud’s built-in functionality.
HCL
terraform {
  required_version = ">= 0.12.6"
  backend "remote" {
    organization = "<org-name>"
    workspaces {
      name        = "my-workspace"
      prefix      = "<env-prefix>"
    }
  }
}
For the S3 backend, you might need to implement custom versioning using lifecycle policies or separate state files for different versions.

Sync Mechanisms:

Manually sync state files between environments if needed.
Use scripts or CI/CD pipelines to automate this process.
SH
# Example script to sync S3 state with Terraform Cloud
AWS_ACCESS_KEY_ID=<access-key> \
AWS_SECRET_ACCESS_KEY=<secret-key> \
terraform init -backend-config="bucket=my-bucket" \
              -backend-config="region=us-west-2" \
              -backend-config="key=path/to/state/my-workspace.tfstate"

#### Closing Summary

In summary, managing consistency between Terraform Cloud and a self-managed S3 backend requires careful configuration of backends, ensuring state file integrity, implementing version control, and setting up sync mechanisms. By following these steps, you can maintain a consistent and reliable infrastructure across different environments.

#### Common Mistakes/Follow-Ups
Misconfigurations: Incorrectly configuring backend settings leading to data loss or inconsistencies.
Data Loss: Failing to implement proper backup strategies for the state files.
Manual Sync Issues: Relying solely on manual sync without automation, which can lead to errors and delays.

If you have more specific questions about any of these areas, feel free to ask!

**Code:**

```hcl
# Example Terraform Cloud backend configuration
terraform {
  required_version = ">= 0.12.6"
  backend "remote" {
    organization = "<org-name>"
    workspaces {
      name = "my-workspace"
    }
  }
}

# Example S3 backend configuration
terraform {
  required_version = ">= 0.12.6"
  backend "s3" {
    region                  = "us-west-2"
    bucket                  = "<bucket-name>"
    key                     = "path/to/state/my-workspace.tfstate"
    dynamodb_table          = "<dynamodb-table>"
    encrypt_key             = false
    encrypt_with            = "<kms-key-id>"
  }
}
```

