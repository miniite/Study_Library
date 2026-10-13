---
type: "practice"
difficulty: "Expert"
---

# Design the delivery architecture for a system-of-systems where 12 teams' services must be released in a specific dependency order for a major platform migration; how do you encode cross-team ordering, gate on readiness, and recover if one team's step fails midway?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Safe Delivery Platform Changes](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/Safe_Delivery_Platform_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an effective CI/CD delivery architecture for a system-of-systems with 12 teams' services that must be released in a specific dependency order during a major platform migration, we need to address several key aspects: encoding cross-team ordering, ensuring gate readiness, and implementing robust recovery mechanisms. This solution will leverage DevOps best practices, such as semantic versioning, declarative configurations, and automated deployment pipelines.

The core idea is to create a hierarchical and modular CI/CD pipeline that can handle dependencies between teams while providing clear feedback loops for team readiness and failure recovery. Here's the step-by-step approach:

Understand the Question: The primary goal is to ensure that services are released in the correct order, gates on readiness conditions (like tests passing or infrastructure availability), and recovers gracefully if any service fails during deployment.

#### Assumptions

Each team has its own Git repository.
Teams use semantic versioning for their releases.
Infrastructure-as-Code (IaC) is used to manage cloud resources.
Continuous Integration/Continuous Deployment (CI/CD) tools are in place, like Jenkins, GitLab CI, or CircleCI.

#### First Principles Solution

Hierarchical Pipelines: Create a master pipeline that orchestrates the release of all 12 services in order. Each service's pipeline will be nested within this master pipeline.
Semantic Versioning and Dependency Tracking: Use version numbers to track dependencies between teams. For example, if Service A depends on Service B, then any update to Service B should increment its version number.
Readiness Checks: Implement automated checks before each deployment step. These could include unit tests, integration tests, security scans, and infrastructure health checks.
Rollback Mechanisms: Define a clear rollback strategy in case of failures during the deployment.

Detailed Implementation:

Master Pipeline Configuration: Use YAML or JSON to define the master pipeline that includes all services' pipelines as stages.

YAML
stages:
  - pre-deployment-checks
  - deployments
  - post-deployment-verification

jobs:
  - name: check-team-A-readiness
    stage: pre-deployment-checks
    steps:
      - script: |
          # Check if Team A's service is ready for deployment
          echo "Checking readiness of Service A..."
          curl http://service-a/health | grep 'UP'

  - name: deploy-team-A-service
    stage: deployments
    needs: [check-team-A-readiness]
    steps:
      - script: |
          # Deploy Service A using IaC templates and CI/CD artifacts
          echo "Deploying Service A..."
          terraform apply

  - name: verify-deployment-of-team-A-service
    stage: post-deployment-verification
    needs: [deploy-team-A-service]
    steps:
      - script: |
          # Verify the deployment of Service A
          echo "Verifying Service A..."
          curl http://service-a/health | grep 'UP'

  ... (similar blocks for other teams)

Cross-Team Dependencies: Use environment variables or configuration files to pass dependency information between pipelines. For example, SERVICE_DEPENDENCIES can be a YAML file that lists dependencies.

YAML
service-dependencies:
  A: [B, C]
  B: [D, E]
  ...

Automated Rollback: Implement rollback steps in each job's failure condition. This could involve reverting infrastructure changes or redeploying previous versions.

YAML
jobs:
  - name: deploy-team-A-service
    stage: deployments
    needs: [check-team-A-readiness]
    on_failure:
      script: |
        # Rollback deployment if it fails
        echo "Rolling back Service A deployment..."
        terraform destroy

#### Closing Summary
and Common Mistakes/Follow-Ups:

Ensure the master pipeline is well-documented, so new team members can understand how to integrate their services.
Regularly review and update the dependency tracking mechanism as teams evolve or add new services.
Use monitoring tools like Prometheus and Grafana to track overall system health during migrations.

**Code:**

```yaml
# Example master pipeline YAML
stages:
  - pre-deployment-checks
  - deployments
  - post-deployment-verification

jobs:
  - name: check-team-A-readiness
    stage: pre-deployment-checks
    steps:
      - script: |
          # Check if Team A's service is ready for deployment
          echo "Checking readiness of Service A..."
          curl http://service-a/health | grep 'UP'

  - name: deploy-team-A-service
    stage: deployments
    needs: [check-team-A-readiness]
    steps:
      - script: |
          # Deploy Service A using IaC templates and CI/CD artifacts
          echo "Deploying Service A..."
          terraform apply

  - name: verify-deployment-of-team-A-service
    stage: post-deployment-verification
    needs: [deploy-team-A-service]
    steps:
      - script: |
          # Verify the deployment of Service A
          echo "Verifying Service A..."
          curl http://service-a/health | grep 'UP'

  ... (similar blocks for other teams)
```

**Example input / output:**

```text
No specific input-output examples are required for this high-level architecture. However, the example above provides a clear structure that can be used as a template.
```

**Data:**

```yaml
# Example service dependencies YAML
service-dependencies:
  A: [B, C]
  B: [D, E]
  ...
```

