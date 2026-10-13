---
type: "practice"
difficulty: "Expert"
---

# How do you architect a delivery platform to serve both a two-pizza startup pace and a regulated banking division within the same company, where the same platform enforces heavy controls for one and stays lightweight for the other?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Safe Delivery Platform Changes](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/Safe_Delivery_Platform_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to architect a CI/CD delivery platform that can serve both a fast-moving two-pizza startup pace and a heavily regulated banking division within the same organization. The challenge is to ensure that the platform enforces strict controls for the banking division while remaining lightweight and flexible for the startup.

The key idea here is to design a modular, multi-tenant architecture where we can dynamically apply different levels of security, compliance, and performance settings based on the tenant's requirements. This involves leveraging multi-environment configurations, policy-driven access control, and dynamic feature toggling.

#### Triage
Question: How do you design a CI/CD platform that caters to both high-velocity development for a startup and strict regulatory controls for a banking division?
What is being probed:
Understanding of multi-environment architectures.
Knowledge of policy-driven access control and dynamic feature toggling.
Experience with compliance requirements in regulated industries.

#### Assumptions
The platform must support different levels of security, performance, and governance depending on the division.
Step-by-Step Solution

Multi-Tenancy Architecture:

Implement a multi-tenancy model where each tenant (startup vs banking) has its own isolated environment.
Use namespaces or separate Docker images to ensure isolation between tenants' codebases.

Dynamic Feature Toggling:

Leverage feature flags to enable or disable features based on the tenant. This allows the platform to offer different functionalities without requiring full redeploys for compliance reasons.

Policy-Driven Access Control:

Implement role-based access control (RBAC) and policy-as-code to enforce strict controls in the banking division.
Use tools like OAuth, OpenID Connect, or custom policies to manage user roles and permissions dynamically.

Environment Configuration Management:

Use environment-specific configurations for different stages of deployment (development, testing, production).
Ensure that sensitive data such as secrets are managed securely using tools like HashiCorp Vault or AWS Secrets Manager.

Compliance-First Design:

Follow industry best practices and regulatory requirements specific to the banking division.
Regularly audit and ensure compliance with relevant laws and regulations (e.g., GDPR, PCI-DSS).

#### Closing Summary

By adopting a multi-tenant architecture, dynamic feature toggling, policy-driven access control, and environment-specific configuration management, we can deliver a CI/CD platform that meets the diverse needs of both high-velocity startups and heavily regulated banking divisions. This approach ensures flexibility for rapid development while maintaining robust security and compliance standards.

#### Common Mistakes/Follow-Ups
Lack of Isolation: Not properly isolating tenants' environments, leading to accidental data leakage.
Overcomplication: Over-engineering the solution with unnecessary complexity that may slow down deployment cycles.
Ignoring Compliance: Failing to implement adequate compliance measures, putting the organization at risk.

**Comparison:**

```
Tenant	Features Enabled	Access Control Policies
Startup	[featureA, featureB]	None
Banking	[]	User1: read+write; User2: read-only
```

**Code:**

```python
# Example of dynamic feature toggling using a simple dict
features = {
    "startup_mode": True,
    "banking_compliance": False  # Default to False for startup
}

if features["banking_compliance"]:
    # Implement compliance checks and controls here
    pass
else:
    # Startup-specific logic
    pass
```

**Example input / output:**

```text
Input:
- Tenant: startup
- Feature Request: featureA

Output:
featureA is enabled for tenant 'startup'

Input:
- Tenant: banking
- Feature Request: featureC

Output:
featureC is not available for tenant 'banking'
```

**Data:**

```json
{
  "tenants": {
    "startup": {
      "namespace": "prod-startup",
      "features": ["featureA", "featureB"]
    },
    "banking": {
      "namespace": "prod-banking",
      "features": [],
      "access_policies": {
        "user1": ["read", "write"],
        "user2": ["read-only"]
      }
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. CI/CD Pipeline
4. Is Tenant 'Banking'?
5. Apply Compliance Policies
6. Deploy to 'prod-banking'
7. Enable Startup Features
8. Deploy to 'prod-startup'

