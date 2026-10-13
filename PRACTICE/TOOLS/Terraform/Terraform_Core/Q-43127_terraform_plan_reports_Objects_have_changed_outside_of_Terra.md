---
type: "practice"
difficulty: "Intermediate"
---

# terraform plan reports 'Objects have changed outside of Terraform.' Walk me through how you decide to accept the drift versus revert it.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Refactoring with State Moves](<../../../../TOOLS/Terraform/Terraform_Core/Refactoring_with_State_Moves.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a DevOps environment using Terraform, encountering state drift notifications can be challenging but crucial to manage effectively. When Terraform reports "Objects have changed outside of Terraform," it means that resources managed by Terraform have been modified manually or via other tools without updating the Terraform state file. The key decision is whether to accept this change and update the Terraform state, or revert the changes to maintain consistency with your Terraform configuration.

Let's break down the process step-by-step:

Understand the Context: First, identify which resources are affected by the drift. For example, if a network interface on an EC2 instance has been manually modified in AWS Console, this will cause drift.

Evaluate Impact and Purpose of Change:

Is the change critical for your environment? For instance, a security group rule added via the console might be necessary.
Does it align with your current Terraform configuration or desired state? If the manual changes are required to meet specific business needs, they should be integrated.

Check Automation and Version Control: Ensure that all Terraform code is committed to version control (e.g., Git). This helps in tracking changes and ensuring consistency across environments.

Update Terraform State:

Use terraform import or terraform state replace-module to incorporate these manual changes into your Terraform state file.
Re-run terraform plan to see the changes that will be applied.

Review and Test Changes: Manually test the configuration to ensure that the drift does not cause issues in production.

Document and Communicate:

Document why certain manual changes were necessary and how they align with your infrastructure goals.
Communicate these decisions to relevant stakeholders to maintain transparency.

Implement Controls: Consider implementing strategies to prevent future drift, such as using Terraform's terraform taint command for resources that should not be modified manually or setting up continuous integration/continuous deployment (CI/CD) pipelines that enforce consistency with your infrastructure-as-code files.

By following these steps, you can make an informed decision on whether to accept the drift and update your Terraform state or revert it. This process ensures that your infrastructure remains in sync with your configuration and minimizes manual intervention.

