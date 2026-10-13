---
type: "practice"
difficulty: "Expert"
---

# How would you make terraform plan on a PR trustworthy as a review artifact, and what stops a plan-then-apply race where infra changes between the two steps?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Test Data Governance](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Validating_Pipeline_Changes/Test_Data_Governance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that Terraform plans generated from Pull Requests (PRs) are trustworthy as a review artifact, we need to integrate a robust CI/CD pipeline that ensures consistency and correctness before applying changes to production infrastructure. The core idea is to create a seamless workflow where the PR build checks the plan against expected states without affecting the live environment.

The key steps involve:

Generate a Terraform Plan: Run terraform plan locally or in a dedicated CI/CD stage.
Validate the Plan: Ensure that the plan adheres to predefined validation rules and best practices, such as cost constraints, compliance checks, and resource uniqueness.
Store the Plan: Save the validated plan as an artifact in your CI system for review.
Integrate with PR Reviews: Use pull request tools to attach and discuss these artifacts.

Now let's break down each step from first principles:

Step-by-Step Solution
1. Generate a Terraform Plan

Running terraform plan on the codebase will generate a list of changes that would be applied if terraform apply were executed. This can be done either locally by contributors or in CI/CD stages.

BASH
# Example command to run terraform plan
terraform plan -out=tfplan
2. Validate the Plan

The generated plan should then undergo validation checks. This can include static code analysis tools, custom scripts, or integration with other infrastructure-as-code management systems like Pulumi or Ansible.

BASH
# Example validation script to check for cost overruns
./validate_plan.sh tfplan > validation_report.txt
3. Store the Plan as an Artifact

The validated plan can be stored in a CI/CD system such as Jenkins, GitLab CI, GitHub Actions, or GitOps platforms like Argo CD. This ensures that the plan is version-controlled and traceable.

YAML
# Example GitLab CI configuration to store validation report
stages:
  - validate

validate_plan:
  stage: validate
  script:
    - terraform plan -out=tfplan
    - ./validate_plan.sh tfplan > validation_report.txt
  artifacts:
    paths:
      - validation_report.txt
4. Integrate with PR Reviews

The validated plan and its associated reports should be linked to the corresponding PR in your version control system. This allows reviewers to inspect the changes before they are applied.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, ensuring a trustworthy Terraform plan as part of CI/CD involves generating plans, validating them, storing them securely, and integrating the results into PR reviews. Common mistakes include skipping validation steps or failing to store artifacts properly. Follow-ups might involve enhancing the validation process with machine learning models for automated cost estimation or compliance checks.

#### Triaging the Question

The interviewer is probing your understanding of:

How CI/CD integrates with infrastructure-as-code (IaC) tools.
Best practices for ensuring consistency and correctness in IaC changes.
Strategies to prevent race conditions between plan generation and apply operations.

Your ability to articulate a comprehensive solution demonstrates your expertise in both practical implementation and strategic thinking around CI/CD pipelines.

Depth + Language Match the Level

For an expert-level answer, focus on deep insights into infrastructure management and CI/CD best practices. Provide real-world examples and discuss trade-offs between automation and manual review processes.

**Code:**

```bash
# Example command to run terraform plan
terraform plan -out=tfplan

# Example validation script to check for cost overruns
./validate_plan.sh tfplan > validation_report.txt
```

**Flow:**

1. Generate Terraform Plan
2. Validate Plan
3. Store as Artifact
4. Integrate with PR Reviews

