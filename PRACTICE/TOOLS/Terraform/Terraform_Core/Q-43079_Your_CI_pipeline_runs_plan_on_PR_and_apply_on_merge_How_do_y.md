---
type: "practice"
difficulty: "Deep"
---

# Your CI pipeline runs plan on PR and apply on merge. How do you make sure the applied plan matches the reviewed plan given state may have changed in between?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reviewing Plans for State Safety](<../../../../TOOLS/Terraform/Terraform_Core/Reviewing_Plans_for_State_Safety.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that the apply step of your Terraform pipeline accurately reflects the reviewed changes, we need to manage and understand the state file effectively while leveraging Terraform modules. Here's a deep dive into how you can achieve this:

TL;DR
Understand State Management: Ensure that the state file is properly managed and stored in a version control system.
Use Modules for Reusability and Flexibility: Organize your infrastructure code into reusable Terraform modules to maintain consistency.
Plan/Apply Flow: Implement a CI pipeline where plan is run on PR, and apply is executed post-merge.

#### Triaging the Question

The question revolves around ensuring that the applied changes match the reviewed changes in a CI pipeline scenario involving Terraform state management. The core concern here is managing the state file's consistency between the plan stage (PR) and apply stage (post-merge).

#### Assumptions
State Management: The state file is stored securely, typically in an S3 bucket or backend database.
CI Pipeline Integration: CI tools like GitLab, Jenkins, or GitHub Actions are being used to automate the pipeline.
Terraform Version Control: Terraform code and state files are version-controlled alongside your application code.
Step-by-Step Solution
1. Managing State Consistency

To ensure that the state file remains consistent between plan and apply stages:

Store State Securely: Use a backend for Terraform state, like S3 or RDS, to store the state securely.
Version Control Integration: Integrate your CI tool with version control (e.g., Git) to track changes in both code and state.
2. Using Modules for Reusability

Terraform modules are a powerful way to manage infrastructure configurations:

Organize Code: Break down complex Terraform configurations into smaller, reusable modules.
Parameterization: Use variables and inputs to parameterize your modules, making them more flexible.
3. Implementing the CI Pipeline

Your CI pipeline should have two stages: plan on PR and apply post-merge:

PR Stage (Plan): Run a terraform plan command within the CI tool to generate a diff report that can be reviewed.
Merge Stage (Apply): Post-merge, run terraform apply using the updated state file.
Sample Workflow

Code Commit and PR:

BASH
# Example commit message
git commit -m "Add new VPC module"

# Push to trigger CI
git push origin feature/vpc-update

CI Plan Stage (PR):

YAML
stages:
  - plan

plan-stage:
  stage: plan
  script:
    - terraform init
    - terraform plan
  artifacts:
    paths:
      - "terraform.plan"

Merge and CI Apply Stage:

YAML
stages:
  - apply

apply-stage:
  stage: apply
  when: on_success
  script:
    - terraform init -backend-config="bucket=<s3-bucket-name>"
    - terraform apply -auto-approve

#### Closing Summary
and Common Mistakes

Common Mistake: Not versioning the state file alongside code.

Fix: Ensure that both your Terraform code and state files are stored in a Git repository.

Common Mistake: Running terraform apply without reviewing diffs in PR.

Fix: Implement static analysis tools or manual reviews to catch potential issues before applying changes.

Common Mistake: Ignoring the backend configuration for state management.

Fix: Ensure that you always use a backend for storing Terraform state, whether it's S3, RDS, or any other provider.
Final Check

I have covered all aspects of ensuring consistent state between plan and apply stages in a CI pipeline. The solution includes managing the state securely, using modules for reusability, and implementing a robust CI flow with artifact tracking.

**Comparison:**

```
Step	Action
1.	Commit code and trigger PR in CI tool.
2.	Run terraform plan to generate diffs.
3.	Review diffs and merge changes into master.
4.	Run terraform apply -auto-approve post-merge.
```

**Code:**

```yaml
# Example CI/CD YAML
stages:
  - plan
  - apply

plan-stage:
  stage: plan
  script:
    - terraform init
    - terraform plan
  artifacts:
    paths:
      - "terraform.plan"

apply-stage:
  stage: apply
  when: on_success
  script:
    - terraform init -backend-config="bucket=<s3-bucket-name>"
    - terraform apply -auto-approve
```

**Example input / output:**

```text
# Example Plan Output
$ terraform plan
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be persisted to the local state file.

------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # module.vpc.aws_vpc.main will be created
  + resource "aws_vpc" "main" {
      + arn                    = (known after apply)
      + id                     = (known after apply)
      + tags                   = {
          + "Name" = "main-vpc"
        }
      + vpc_id                 = (known after apply)

      + cidr_block             = "10.0.0.0/16"
    }

Plan: 1 to add, 0 to change, 0 to destroy.

# Example Apply Output
$ terraform apply -auto-approve
aws_vpc.main: Creating...
```

**Data:**

```json
{
  "state_bucket": "<s3-bucket-name>",
  "terraform_backend_config": {
    "bucket": "<s3-bucket-name>",
    "key": "path/to/statefile.tfstate",
    "region": "us-west-2"
  }
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. A[Code Commit] --> B[Trigger CI/CD Pipeline];
4. B --> C[PR Stage (Plan)]{Run `terraform plan`};
5. C --> D[Review Changes]{Ensure diffs match PR review};
6. D --> E[Merge to Master]{Post-merge changes};
7. E --> F[Apply Stage (Apply)]{Run `terraform apply -auto-approve`};

