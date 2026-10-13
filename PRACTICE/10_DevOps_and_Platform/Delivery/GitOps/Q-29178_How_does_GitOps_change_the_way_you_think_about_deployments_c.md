---
type: "practice"
difficulty: "Basic"
---

# How does GitOps change the way you think about deployments compared to SSHing into servers?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ GitOps Fundamentals](<../../../../10_DevOps_and_Platform/Delivery/GitOps/GitOps_Fundamentals/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary: GitOps fundamentally shifts how DevOps teams manage infrastructure and application states by ensuring that both the production environment and code repositories are in sync, leading to more reliable and automated deployments.

#### Triaging the Question
The core of this question is about understanding the shift from manually managing server configurations via SSH (i.e., "infrastructure as a script") to a GitOps approach where infrastructure is managed by version-controlled configuration files. This involves how GitOps ensures consistency, transparency, and automation in the deployment process.

#### First-Principles Solution

Manual vs. Automated: Traditionally, operations teams might manually SSH into servers to make changes to configurations or apply new software updates. This approach is error-prone, time-consuming, and lacks a version control system.

Infrastructure as Code (IaC): GitOps leverages IaC tools like Terraform, Ansible, or Kubernetes manifests stored in version-controlled repositories such as GitHub or GitLab. These files define the desired state of infrastructure, ensuring that every change is documented and traceable.

Continuous Integration/Deployment: In a GitOps workflow, changes are pushed to these repositories, triggering CI/CD pipelines. This automation ensures that the production environment mirrors what’s defined in the repository, reducing human error and increasing reliability.

Observability and Traceability: With GitOps, each change is recorded in version control, making it easy to track when and by whom changes were made. This transparency helps in debugging issues more effectively and maintaining a historical record of all modifications.

Self-Healing Systems: Modern GitOps frameworks can automatically detect and correct discrepancies between the desired state (defined in the repository) and the current state (the actual infrastructure). For example, Kubernetes uses self-healing mechanisms to ensure that the cluster always matches its configuration file.

Reversibility and Rollbacks: If a change introduces issues, GitOps allows for quick rollbacks by reverting changes in the version-controlled repository. This is far more efficient than trying to undo manual server edits through SSH.

#### Closing Summary
GitOps transforms deployment processes from ad-hoc, error-prone SSH sessions into systematic, version-controlled workflows that ensure consistency and reliability. It democratizes infrastructure management, making it easier for developers and operations teams to collaborate effectively.

#### Common Mistakes/Follow-ups

Misunderstanding the Automation: A common mistake is thinking that GitOps eliminates human intervention entirely. Instead, it shifts the focus from manual SSH sessions to a more systematic approach.
Ignoring Observability: Teams sometimes overlook the importance of logging and monitoring in GitOps. Proper observability tools are crucial for understanding how changes affect the system.
Overcomplicating Changes: Overzealous teams might introduce complex workflows that make change management harder rather than easier.

**Comparison:**

```
Manual SSH	GitOps
High risk of human error	Reduces risk through automation and version control
Hard to track and trace history	Provides a clear audit trail
```

**Code:**

```yaml
# Example Terraform configuration file
resource "aws_instance" "example" {
  ami           = var.ami_id
  instance_type = var.instance_type
}
```

**Example input / output:**

```
Example Git commit and deployment process:
PLAINTEXT
Copy
$ git add .
$ git commit -m "Update instance type"
$ git push origin master

# CI/CD Pipeline Triggered, changes deployed to production.
```

**Data:**

```
Repository structure:
CODE
Copy
/repos/terraform/
  - main.tf
  - variables.tf
  - outputs.tf
```

**Flow:**

1. No
2. Yes
3. Manual Change
4. Is the change tracked?
5. Made via SSH
6. Pushed to Git Repository
7. CI/CD Pipeline TriggersDeployment
8. Production EnvironmentUpdated

