---
type: "practice"
difficulty: "Deep"
---

# How would you design a module registry versioning strategy (semver, ~> constraints) so teams get patches automatically but never surprise majors?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Module Compatibility](<../../../../TOOLS/Terraform/Terraform_Core/Module_Compatibility.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of designing a robust module registry versioning strategy in Terraform that ensures teams receive automatic patch updates while avoiding unexpected major version changes, we need to focus on leveraging semantic versioning (SemVer) and the ~> constraint syntax effectively.

#### Triage

The question is probing our understanding of how to manage dependencies within Terraform modules using version constraints. Specifically, it requires us to implement a versioning strategy that allows teams to get minor and patch updates automatically but prevents them from receiving major updates accidentally.

We need to make several assumptions:

The module registry (e.g., TF Registry) has different versions of the same module available.
Teams are using Terraform modules across multiple projects or environments.
We aim for a strategy that is easy to manage and understand, with minimal manual intervention required.

#### First-Principles Solution

To achieve this, we can use SemVer (Major.Minor.Patch) along with the ~> constraint syntax in our Terraform configuration files. The ~> operator allows us to specify that we want all minor and patch updates but block major versions. For example, if a module has version 1.2.0, using module "example" { source = "path/to/module" version = "~> 1.2.0" } would allow us to get any subsequent minor or patch releases (e.g., 1.2.1, 1.2.2) but prevent any major updates (e.g., 2.0.0).

#### Implementation
Steps
Define Versioning Strategy: Use SemVer for all module versions.
Use ~> Constraints: Ensure that the version constraints are set to allow minor and patch updates only.
Automate Updates: Use a CI/CD pipeline or a separate Terraform script to periodically check for available updates and apply them.

#### Example
Configuration

Here’s an example of how you might configure a module in your Terraform code:

HCL
module "example" {
  source = "path/to/module"
  version = "~> 1.2.0"
}

To ensure teams receive patches automatically but never unexpected major updates, we can implement the following steps:

Automate Version Check: Write a script to periodically check for new versions of the module and apply them.
Notify Teams: Set up notifications to inform team members about available updates so they can review and approve before applying.

#### Closing Summary

By using SemVer with the ~> constraint in Terraform, we ensure that teams receive automatic minor and patch updates but are protected from unexpected major version changes. This approach balances flexibility with safety by allowing for controlled updates while maintaining stability.

#### Common Mistakes/Follow-Ups
Incorrect Versioning: Using > or < instead of ~> can lead to unintended version mismatches.
Manual Updates: Not automating the update process may result in manual errors and delays.
Overly Permissive Constraints: Allowing too many versions might introduce breaking changes unintentionally.

To avoid these mistakes, it's crucial to carefully define and document the versioning strategy, ensuring that all team members understand how updates are managed.

**Comparison:**

```
Table of module versions and their purposes:
Version	Purpose
v1.1.0	Stable, production-ready
v1.2.0	Minor update with new features but no breaking changes
v1.2.1	Patch fixing bugs in minor version
```

**Code:**

```hcl
module "example" {
  source = "path/to/module"
  version = "~> 1.2.0"
}
```

**Example input / output:**

```text
Input:
terraform state list | grep "example" | awk '{print $4}' | sed 's/version=//'
terraform state replace-module example path/to/module~>1.2.0

Output:
1.1.0

New version available: 1.2.0
```

**Data:**

```

Sample module version history:

v1.1.0 (stable)
v1.2.0 (minor update with new features)
v1.2.1 (patch fixing bugs)

Sample Terraform configuration for checking updates:

BASH
Copy
#!/bin/bash

# Function to check for updated module versions
check_module_version() {
  current_version=$(terraform state list | grep "example" | awk '{print $4}' | sed 's/version=//')

  latest_version=$(curl -s https://api.example.com/modules/example | jq -r '.versions[-1].version')

  if [[ "$current_version" != "$latest_version" ]]; then
    echo "New version available: $latest_version"
    # Apply the update using Terraform
    terraform state replace-module example path/to/module~>"$latest_version"
  else
    echo "No new versions available."
  fi
}

check_module_version
```

**Flow:**

1. Yes
2. No
3. Check Module Version
4. Current Version = LatestVersion?
5. No New Versions Available
6. Notify Teams about Update
7. Apply Update using Terraform

