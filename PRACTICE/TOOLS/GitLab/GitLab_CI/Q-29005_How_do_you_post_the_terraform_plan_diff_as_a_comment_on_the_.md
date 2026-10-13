---
type: "practice"
difficulty: "Intermediate"
---

# How do you post the terraform plan diff as a comment on the merge request for review?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ GitLab CI Fundamentals](<../../../../TOOLS/GitLab/GitLab_CI/GitLab_CI_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address how to post a Terraform plan diff as a comment on a GitLab merge request, we need to ensure that CI/CD pipelines are set up to run Terraform plans and capture their outputs automatically. The core idea is to leverage the gitlab-ci.yml configuration file within your project to define stages for running Terraform commands and then use GitLab’s API or webhooks to post these diffs as comments on merge requests.

#### Triage

The question asks how to integrate a CI/CD pipeline with Terraform to automatically post plan diffs as comments on GitLab merge requests. This involves understanding the workflow, configuration of gitlab-ci.yml, and utilizing GitLab APIs for commenting.

#### Assumptions
You have an existing GitLab project with a .gitlab-ci.yml file.
Your team is using Terraform to manage infrastructure as code.
The merge request process in GitLab is active and functional.

#### Solution

The solution involves setting up a CI/CD pipeline that runs terraform plan to generate the diff, captures the output, and then posts it as a comment on the merge request. Here’s how you can achieve this:

Define Stages in .gitlab-ci.yml: Configure your GitLab CI job to run Terraform commands.
Capture Output of terraform plan: Use tee or similar tools to redirect output to a file that can be included as part of the pipeline job.
Post Comment Using GitLab API: Write a script to post the diff as a comment on the merge request using the GitLab API.
Step-by-Step Solution

Define Stages in .gitlab-ci.yml:

YAML
stages:
  - plan
  - review

terraform-plan:
  stage: plan
  script:
    - terraform init
    - terraform plan > terraform_plan.out  # Redirect output to a file
    - cat terraform_plan.out              # Optionally print the diff on CI console

Capture Output of terraform plan: The above example captures the output of terraform plan and stores it in a file named terraform_plan.out. This can be further processed if needed.

Post Comment Using GitLab API: After generating the diff, you need to post this information as a comment on the merge request. You can use the following Python script (with necessary imports and authentication):

PYTHON
import requests
from gitlab import Gitlab

gl = Gitlab('https://gitlab.example.com', private_token='your_private_token')

project_id = 123456  # Replace with your project ID
merge_request_id = 7890  # Replace with your MR ID
diff_output_path = "terraform_plan.out"  # Path to the file containing the Terraform plan output

with open(diff_output_path, 'r') as f:
    diff_content = f.read()

mr = gl.projects.get(project_id).mergerequests.get(merge_request_id)
mr.notes.create({'body': diff_content})

#### Closing Summary

By configuring your GitLab CI pipeline to run Terraform plans and using the GitLab API, you can automatically post the diff as a comment on merge requests. This ensures that team members reviewing changes can quickly see what changes will be made before merging.

#### Common Mistakes/Follow-Ups
Mistake: Not properly handling sensitive information in scripts or not setting up the correct authentication.

#### Follow-Up
Ensuring the script is idempotent and robust against errors, such as network issues when posting comments.
Mistake: Failing to test the CI pipeline thoroughly before deploying it.

#### Follow-Up
Setting up continuous testing of your CI configurations in a staging environment before production deployment.

**Code:**

```

stages:

plan
review

terraform-plan: stage: plan script: - terraform init - terraform plan > terraform_plan.out - cat terraform_plan.out

CODE
Copy

```python
import requests
from gitlab import Gitlab

gl = Gitlab('https://gitlab.example.com', private_token='your_private_token')

project_id = 123456
merge_request_id = 7890
diff_output_path = "terraform_plan.out"

with open(diff_output_path, 'r') as f:
    diff_content = f.read()

mr = gl.projects.get(project_id).mergerequests.get(merge_request_id)
mr.notes.create({'body': diff_content})
```

**Data:**

```yaml
stages:
  - plan
  - review

terraform-plan:
  stage: plan
  script:
    - terraform init
    - terraform plan > terraform_plan.out
    - cat terraform_plan.out
```

**Flow:**

1. CI Job
2. Terraform Init
3. Terraform Plan >terraform_plan.out
4. Post Diff to MR Comment (API)

