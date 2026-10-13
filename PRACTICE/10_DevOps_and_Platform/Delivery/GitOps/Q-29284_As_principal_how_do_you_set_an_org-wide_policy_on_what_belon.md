---
type: "practice"
difficulty: "Deep"
---

# As principal, how do you set an org-wide policy on what belongs in git versus what is legitimately runtime-owned state, and how do you enforce that boundary architecturally?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Multi-cluster Ordering](<../../../../10_DevOps_and_Platform/Delivery/GitOps/GitOps_Fundamentals/Multi-cluster_Ordering.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Setting an org-wide policy on GitOps boundaries between code and runtime state requires a holistic approach that balances flexibility with security and compliance. The key is to clearly define which components should be managed via GitOps and which can safely reside in the runtime environment. Here’s how you would architect this:

Define Clear Boundaries: Establish what constitutes code versus runtime state.
Enforce Policies: Implement mechanisms to ensure adherence to these boundaries.
Monitor and Audit: Continuously monitor and audit compliance.

Let's dive into each step in detail.

#### Triaging the Question

The core of this question is about delineating responsibilities between the GitOps-controlled environment and runtime state, ensuring that sensitive data or configurations are not mistakenly stored in code repositories. The interviewer is probing your understanding of security, governance, and technical implementation strategies.

#### Assumptions
The organization has a mature CI/CD pipeline.
There are existing tools for monitoring and alerting on Git changes.
Runtime state includes databases, secrets, configuration files, etc., that should not be stored in version control systems (VCS).

#### First Principles Solution

Define the Boundaries:

Code Repository: Contains application source code, infrastructure as code (IaC), and deployment scripts.
Runtime State: Includes database schemas, secrets, configuration files, logs, etc.

GitOps Workflow:

GitOps ensures that the desired state of your systems is managed in code repositories, which are then reconciled with actual runtime states.

Policy Enforcement:

Use tools like git-secrets to prevent sensitive data from being committed.
Implement CI/CD pipelines that automatically check for violations before merging changes into main branches.
Utilize infrastructure-as-code (IaC) tools like Terraform or Ansible to manage runtime state.

Monitoring and Auditing:

Set up continuous monitoring using tools like GitLab’s built-in security features or third-party solutions such as Trivy.
Implement regular audits with automated checks for compliance.
Architectural Design

To architect this effectively, we can break it down into several layers:

Code Repository Layer:

Code Repository
IaC
CI/CD
SecretsManager

Runtime State Layer:

Runtime State
DB
Monitoring

Governance Layer:

CodeRepo
PolicyEnforcement
CI/CD
SecretsManager
RuntimeState
Audit

#### Example
Implementation

Code Repository and IaC:

Use Terraform to manage infrastructure.
HCL
resource "aws_s3_bucket" "example_bucket" {
  bucket = "example-bucket"
  acl    = "private"
}

Secrets Management:

Utilize a secrets management tool like HashiCorp Vault or AWS Secrets Manager.
BASH
vault write secret/yourapp password=supersecret

CI/CD Pipeline:

Configure the pipeline to check for sensitive data before merging into main branches.
YAML
steps:
  - name: "Check for Sensitive Data"
    command: git-secrets scan --all-commits
  - name: "Build and Deploy"
    command: ./build_and_deploy.sh

Monitoring and Audit:

Implement regular audits using tools like Trivy to check for vulnerabilities.
BASH
trivy image your-image-name

#### Closing Summary

By defining clear boundaries, enforcing policies through GitOps practices, and continuously monitoring and auditing the system, you can effectively manage the separation of code from runtime state in an org-wide policy. Common mistakes include over-reliance on manual processes or insufficient automation, leading to compliance gaps.

#### Common Mistakes/Follow-Ups
Manual Approvals: Relying too much on human intervention for approvals and checks.
Inadequate Logging: Failing to log actions that breach the boundary, making it hard to track down issues.
Lack of Automation: Not automating security checks, leading to a higher risk of violations.

**Code:**

```

resource "aws_s3_bucket" "example_bucket" { bucket = "example-bucket" acl = "private" }

CODE
Copy

```bash
vault write secret/yourapp password=supersecret
trivy image your-image-name
```

