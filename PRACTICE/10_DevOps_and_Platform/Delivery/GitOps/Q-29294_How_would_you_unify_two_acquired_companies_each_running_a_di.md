---
type: "practice"
difficulty: "Expert"
---

# How would you unify two acquired companies each running a different GitOps stack (one Flux, one Argo CD) into a single governed platform without freezing delivery for either?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Multi-tenant GitOps](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Repo_and_Environment_Layout/Multi-tenant_GitOps.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Unifying two GitOps stacks from acquired companies involves merging their respective operations to create a cohesive and governed platform that both teams can manage effectively. The key challenge is ensuring seamless integration while maintaining operational continuity, especially in environments where continuous delivery is crucial.

#### Triage
Understanding the Question

The question requires us to integrate Flux (one company) and Argo CD (the other) into a unified GitOps stack without disrupting ongoing deployments or development cycles. The goal is to streamline operations, reduce friction, and enhance governance across both teams.

#### Assumptions
Both companies have existing workloads deployed on Kubernetes clusters.
The Flux and Argo CD stacks are managing different sets of applications with varying release cadences.
There’s a need for common policies, such as security scans or compliance checks, to be applied across the unified stack.
Step-by-Step Solution

Assessment and Planning:

Audit Current Configurations: Identify differences in deployment strategies, automation scripts, and governance rules between Flux and Argo CD.
Define Common Grounds: Determine shared policies that both GitOps stacks support, such as resource management, authentication, and authorization.

Strategic Integration:

Unified Configuration Repository: Create a central configuration repository where both Flux and Argo CD can pull their configurations from. Use tools like GitLab, GitHub, or Bitbucket for this purpose.
Shared Workflows: Define common workflows that both systems can execute. For instance, using Jira or any other ticketing system to manage CI/CD pipelines.

Deployment Strategy:

Parallel Deployments: Initially, run parallel deployments from both Flux and Argo CD to ensure there are no immediate conflicts.
Rollout Plan: Develop a phased rollout plan where critical services can be migrated first while non-critical services follow later.

Governance Enhancements:

Role-Based Access Control (RBAC): Implement RBAC to ensure that only authorized personnel can make changes, thereby enhancing security.
Monitoring and Logging: Centralize monitoring and logging for both systems using tools like Prometheus and Grafana to track performance and issues.

Automation and Orchestration:

Pipelines Automation: Use CI/CD pipelines to automate the deployment process, ensuring consistency across all applications.
Rollback Strategies: Establish rollback mechanisms in case of failed deployments or issues.

Training and Documentation:

Provide comprehensive training sessions for both teams on how to use the unified GitOps platform effectively.
Document best practices and common pitfalls related to deployment strategies, monitoring, and troubleshooting.

#### Closing Summary

By carefully assessing current configurations, defining shared governance rules, implementing parallel deployments, enhancing security and monitoring, automating workflows, and providing thorough training, we can successfully unify Flux and Argo CD into a single, governed GitOps platform. This approach ensures minimal disruption to ongoing operations while paving the way for improved efficiency and consistency.

#### Common Mistakes/Follow-Ups
Ignoring Existing Workflows: Failing to consider the existing workflows of both teams might lead to resistance or inefficiencies.
Overcomplicating Governance Rules: Overly strict governance rules can stifle innovation; it’s important to find a balance between security and operational agility.
Lack of Training and Documentation: Insufficient training can result in errors, while poor documentation can hinder troubleshooting and maintenance.

**Comparison:**

| Application | Current Deployment Strategy | Unified Strategy |
|-------------|---------------------------|------------------|
| A           | Flux                      | Flux/Argo CD     |
| B           | Argo CD                   | Flux/Argo CD     |
| C           | Hybrid                    | Argocd           |

**Code:**

```bash
# Example script for setting up a unified configuration repository using GitLab CI
stages:
  - init
  - deploy

init:
  stage: init
  script:
    - git clone https://gitlab.example.com/unified-config-repo.git /path/to/repo

deploy-flux:
  stage: deploy
  image: fluxcd/flux2:v1.30.0
  script:
    - flux bootstrap kubernetes --github-owner=unified-config-repo --github-token=$FLUX_GITHUB_TOKEN

deploy-argo-cd:
  stage: deploy
  image: argoproj/argocd:v2.5.2
  script:
    - argocd app create unified-app --repo https://gitlab.example.com/unified-config-repo.git --path .
```

**Example input / output:**

```text
# Example input for a phased rollout plan
Phase 1: Critical Services
- Deployments: A, B, C
- Rollout Date: Next Monday

Phase 2: Non-Critical Services
- Deployments: D, E, F
- Rollout Date: Next Friday
```

**Data:**

```yaml
# Example YAML for RBAC configuration
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: unified-gatekeeper
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

**Flow:**

1. Initiate Audit
2. Audit Current Configurations
3. Define Shared GovernanceRules
4. Create Unified ConfigurationRepository
5. Parallel Deployments
6. Implement RBAC &Monitoring
7. Automate Workflows
8. Rollout Plan
9. Provide Training

