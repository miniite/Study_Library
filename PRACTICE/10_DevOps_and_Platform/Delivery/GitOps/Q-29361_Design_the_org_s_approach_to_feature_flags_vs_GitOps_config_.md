---
type: "practice"
difficulty: "Deep"
---

# Design the org's approach to feature flags vs GitOps config for behavior changes — where does runtime flagging beat a git commit, and how do you keep both auditable?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Sync Notifications](<../../../../10_DevOps_and_Platform/Delivery/GitOps/GitOps_Fundamentals/Sync_Notifications.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

In this discussion, we'll explore the decision-making process between using feature flags versus GitOps configurations for managing runtime behavior changes in an organization's infrastructure. We'll identify scenarios where feature flags provide a more suitable solution and outline strategies to maintain both approaches' auditability.

#### Triage

The question primarily probes our understanding of when it is advantageous to use feature flags over GitOps configurations, as well as how to ensure both methods can be audited effectively. The key assumptions include the organization's existing infrastructure and policies for managing configuration changes. Feature flags allow for rapid experimentation and controlled releases without code changes, whereas GitOps ensures that all configuration updates are versioned and traceable.

#### First-Principles Solution

Identifying Use Cases for Feature Flags:

Development and Testing: Feature flags can be used to enable or disable features during development and testing phases.
Experimentation and A/B Testing: They facilitate controlled rollouts of new features by targeting specific user groups.
Rollback Mechanism: In case a feature causes issues, it can be quickly disabled without deploying a rollback fix.

When to Use GitOps:

Stable Production Environments: For changes that do not require immediate or granular control and need to be versioned for traceability.
Continuous Integration/Continuous Deployment (CI/CD): Ensuring all configuration changes are automated and can be validated against a baseline.

Maintaining Auditability:

Centralized Feature Flag Management: Use tools like LaunchDarkly or Optimizely to manage feature flags centrally, ensuring logs and history are accessible.
GitOps for Configuration Changes: Implement GitOps practices using tools like Flux or Argo CD to manage infrastructure as code (IaC) files. Ensure all changes are committed to a version control system.
Detailed Implementation

Feature Flag Management:

Feature Request
Development
Code Integration
Testing
Staging
Production Rollout
A/B Testing
Rollback Mechanism
Centralized Feature Flagging: Use a centralized management platform to create and manage feature flags. This ensures that all team members can access the current state of features.
Logging and Monitoring: Implement logging and monitoring for flag toggles to track when and how they are used.

GitOps Configuration Management:

Feature Request
IaC Authoring
Code Review
Commit to GitOps Repository
Infrastructure Deployment
Audit Logs
IaC for Stable Configurations: Use Infrastructure as Code (IaC) tools like Terraform, Ansible, or Kubernetes manifests to define stable configurations.
Automated Deployment Pipelines: Integrate GitOps with CI/CD pipelines using Flux or Argo CD to automatically deploy changes and ensure they are versioned.

#### Common Mistakes
and Follow-Ups
Failure to Audit Feature Flags: Not having a robust logging mechanism for feature flags can lead to issues in tracking their usage.
Over-reliance on Manual Processes: Relying too heavily on manual toggling of features without automation can introduce human errors.
Lack of Centralized Management: Failing to centralize the management of feature flags and IaC files can make it difficult to trace changes.

#### Closing Summary

By carefully evaluating use cases, implementing robust practices for both feature flags and GitOps configurations, and ensuring thorough auditability, we can effectively manage runtime behavior changes in a DevOps environment. This approach balances flexibility with version control and traceability, providing a comprehensive solution for modern infrastructure management.

**Code:**

```yaml
# Example IaC configuration using Terraform
resource "aws_s3_bucket" "example" {
  bucket = "example-bucket"
}
```

**Data:**

```json
{
  "featureFlags": [
    {
      "name": "beta-feature",
      "enabled": true,
      "groups": ["users:group1", "users:group2"]
    }
  ]
}
```

