---
type: "practice"
difficulty: "Intermediate"
---

# How do you review a Terraform PR for state safety? What patterns in a diff make you suspect an unintended destroy or replace?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reviewing Plans for State Safety](<../../../../TOOLS/Terraform/Terraform_Core/Reviewing_Plans_for_State_Safety.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When reviewing a Terraform pull request (PR), ensuring state safety is crucial to avoid unintended resource destruction or replacement, which can lead to significant issues such as loss of data and service downtime. In this context, "state" refers to the terraform.tfstate file that stores information about the current infrastructure configuration. The key patterns to look for in a diff include changes in module configurations, deletions, and unexpected resource creations or replacements.

To review a Terraform PR for state safety effectively:

Understand the Current State: Familiarize yourself with the existing infrastructure by examining the current terraform.tfstate file.
Analyze Module Changes: Review any modifications to modules in the codebase. Modules can introduce significant changes that might not be immediately obvious.
Identify Resource Operations:
Unintended Destroy: Look for resource deletions or replacements marked as destructive, especially when they are not intentional.
Unexpected Replacements: Notice any resources being replaced without explicit updates in the code.
Evaluate Variables and Inputs: Check if variables or inputs have been inadvertently altered, which could lead to unintended state changes.

Let’s break down these steps with an example:

Step-by-Step Review

Understand the Current State

Open terraform.tfstate and note the current infrastructure.
JSON
# Sample terraform.tfstate snippet
{
  "serial": 2,
  "modules": [
    {
      "path": ["root", "web"],
      "resources": {
        "aws_instance.web1": { ... },
        "aws_s3_bucket.bucket1": { ... }
      }
    }
  ]
}

Analyze Module Changes

Check for any module blocks in the PR that might have been modified.
HCL
module "web" {
  source  = "./web"
  version = "~> 1.0"
  # New configuration
  instance_type = "t3.large"
}

Identify Resource Operations

Look for any resource blocks that might be deleted or replaced.
HCL
resource "aws_instance" "web2" {
  count         = 0
  # Previously, it was:
  # count         = 1
  ...
}

resource "aws_s3_bucket" "new_bucket" {
  bucket = lower(local.name)
  acl    = "private"
}

Evaluate Variables and Inputs

Inspect any changes to variables or inputs that might affect the state.
HCL
locals {
  name = "example"
}

# Old configuration
variable "name" {
  default = "old-example"
}
Common Patterns to Watch For

Unintended Destroy

resource { count = 0 } where previously it was count = 1.
Resource deletions without a clear reason, such as removing a backend service.

Unexpected Replacements

Resource definitions with significant differences in properties (e.g., instance type change).
Changes to variables or inputs that affect resource behavior.

#### Closing Summary

Reviewing Terraform PRs for state safety involves understanding the current infrastructure, analyzing module changes, identifying resource operations, and evaluating variable modifications. Key patterns to watch for include unintended destroys and unexpected replacements, which can lead to significant issues if not addressed. By following these steps, you can ensure that your deployments are safe and reliable.

#### Common Mistakes
Failing to understand the current state.
Ignoring subtle changes in resource definitions or variables.
Overlooking module modifications that might affect infrastructure.

#### Follow-Ups
Discuss how to automate state validation using tools like terraform plan.
Explore best practices for version control and merging branches to avoid conflicts.

**Comparison:**

```
Current State	PR Changes
aws_instance.web1	No change
aws_s3_bucket.bucket1	No change
aws_instance.web2	Destroyed (count=0)
```

**Code:**

```hcl
# Example Terraform configuration snippet
module "web" {
  source = "./web"
  version = "~> 1.0"
  instance_type = "t3.large"
}

resource "aws_instance" "web2" {
  count         = 0
  ...
}

resource "aws_s3_bucket" "new_bucket" {
  bucket = lower(local.name)
  acl    = "private"
}
```

**Example input / output:**

```
Old configuration

module "web" { source = "./web" version = "~> 1.0" instance_type = "t2.small" }

resource "aws_instance" "web2" { count = 1 ... }

CODE
Copy

```hcl
# New configuration after PR
module "web" {
  source = "./web"
  version = "~> 1.0"
  instance_type = "t3.large"
}

resource "aws_instance" "web2" {
  count         = 0
  # Previously, it was:
  # count         = 1
}
```

**Data:**

```json
# Sample terraform.tfstate snippet
{
  "serial": 2,
  "modules": [
    {
      "path": ["root", "web"],
      "resources": {
        "aws_instance.web1": { ... },
        "aws_s3_bucket.bucket1": { ... }
      }
    }
  ]
}
```

**Flow:**

1. Open `terraform.tfstate` forcurrent state
2. Analyze module changes
3. Identify resource operations(destroy, replace)
4. Evaluate variables and inputs
5. Review PR for unintendedchanges

