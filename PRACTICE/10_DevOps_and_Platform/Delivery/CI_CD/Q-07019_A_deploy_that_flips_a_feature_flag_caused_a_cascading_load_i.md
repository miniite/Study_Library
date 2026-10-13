---
type: "practice"
difficulty: "Deep"
---

# A deploy that flips a feature flag caused a cascading load increase downstream that the flag owner didn't anticipate; how would you build flag changes into the same risk-gated, observable pipeline flow as code deploys?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Feature Flags and Deploy Risk](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Feature_Flags_and_Deploy_Risk.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to integrate feature flag changes into our existing CI/CD pipeline in a way that ensures they are subject to the same risk-gating and observability mechanisms as code deployments. This involves several steps to ensure both safety and efficiency.

TL;DR:

The key is to design a mechanism where any change to a feature flag triggers a similar pipeline flow as a code deployment, including static analysis, automated tests, staging environment deployments, and thorough monitoring in production.

#### Triage
Understand the Problem: A deploy that flipped a feature flag caused unexpected downstream effects.
What is Being Probed:
Risk-Gating Mechanisms: Ensure changes are thoroughly vetted before rolling out to production.
Observability: Track and monitor the impact of the change in real-time.

#### Assumptions
The existing CI/CD pipeline has a robust deployment strategy with risk-gating mechanisms.
Feature flags can be toggled independently without affecting non-flag-related code.

#### Solution
Identify the Change: When a feature flag is changed, it should trigger a similar set of activities as a code change in the CI/CD pipeline.
Create a Separate but Parallel Workflow:
Introduce a "Flag Change" workflow that runs parallel to the "Code Deployment" workflow.
Static Analysis and Automated Tests:
Run static analysis tools to detect potential issues early.
Execute automated tests to ensure the flag change does not introduce regressions or bugs.
Staging Environment Deployment:
Deploy the updated feature flag configuration to a staging environment where it can be thoroughly tested without affecting users.
Monitoring and Canary Releases:
Implement monitoring tools to track the impact of the new flag state in the staging environment.
Rollout Strategy:
Develop a controlled rollout strategy that gradually introduces the new feature state to production.
Post-Deployment Observability:
Continuously monitor the production environment for any adverse effects.
Detailed Solution:
Step 1: Identify the Change

When a developer changes a feature flag, this action should be detected by our CI/CD system. This can be done through a webhook or direct integration with the configuration management tool used to manage flags.

Flag Changed
Trigger CI/CD Pipeline
Step 2: Static Analysis and Automated Tests

The first step in our pipeline should run static analysis tools like SonarQube or Snyk. This helps catch potential issues early on.

B
Run Static Analysis
Step 3: Staging Environment Deployment

Next, the updated flag configuration is deployed to a staging environment for thorough testing. This can be done via a custom deployment script or an existing CI/CD pipeline step.

C
Staging Deployment
Step 4: Monitoring and Canary Releases

In the staging environment, monitoring tools like Prometheus or Grafana are used to track metrics. A canary release strategy ensures that only a small subset of users is impacted initially. If everything looks good, the flag state can be rolled out to all users.

D
Monitor Staging
Canary Rollout
Step 5: Controlled Rollout Strategy

The rollout strategy should be designed based on user groups or geographic regions. This ensures that any issues are contained and can be rolled back quickly if necessary.

F
Controlled Rollout toProduction
Post-Rollout Monitoring
Step 6: Post-Deployment Observability

Finally, post-rollout monitoring is crucial. Any unexpected behavior should be detected and addressed promptly.

H
Monitor in Production
Troubleshoot Issues ifNecessary

#### Closing Summary

By integrating feature flag changes into our existing CI/CD pipeline with the same risk-gating and observability mechanisms, we can ensure that any changes are thoroughly vetted before being rolled out to production. This approach not only reduces risks but also enhances overall system reliability.

#### Common Mistakes/Follow-Ups
Ignoring Observability: Not having robust monitoring tools in place can lead to undetected issues.
Overlooking Static Analysis and Automated Tests: Failing to run these early steps can result in bugs reaching production.
Inadequate Canary Releases: Without a gradual rollout strategy, unexpected issues can cause significant disruptions.

**Code:**

```
MERMAID
Copy
graph TD;
    A[Flag Changed] --> B[Trigger CI/CD Pipeline]
    B --> C[Run Static Analysis]
    C --> D[Staging Deployment]
    D --> E[Monitor Staging] --> F[Canary Rollout]
    F --> G[Controlled Rollout to Production] --> H[Post-Rollout Monitoring]
    H --> I[Monitor in Production] --> J[Troubleshoot Issues if Necessary]
```

**Flow:**

1. Flag Changed
2. Trigger CI/CD Pipeline
3. Run Static Analysis
4. Staging Deployment
5. Monitor Staging
6. Canary Rollout
7. Controlled Rollout toProduction
8. Post-Rollout Monitoring
9. Monitor in Production
10. Troubleshoot Issues ifNecessary

