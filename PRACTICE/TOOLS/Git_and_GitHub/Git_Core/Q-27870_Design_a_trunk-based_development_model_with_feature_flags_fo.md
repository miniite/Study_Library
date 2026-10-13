---
type: "practice"
difficulty: "Deep"
---

# Design a trunk-based development model with feature flags for a 30-person database team shipping schema changes daily, and enumerate the metrics (migration lead time, rollback rate, drift incidents) you'd track to prove it's healthier than long-lived branches.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Branching Strategies](<../../../../TOOLS/Git_and_GitHub/Git_Core/Branching_Strategies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a 30-person database development team tasked with shipping schema changes daily, implementing a trunk-based development (TBD) model alongside feature flags is crucial. This approach ensures that all developers work on a single main branch, reducing merge conflicts and improving collaboration. The key metrics to track—migration lead time, rollback rate, and drift incidents—are essential for maintaining the health of your development process.

To effectively design this model with feature flags:

#### Overview
We'll adopt a TBD model where all changes are made in the trunk/main branch. Feature flags will allow us to enable or disable these features without merging code into production branches.

#### Triage
The goal is to streamline the development process, minimize merge conflicts, and ensure that feature rollouts can be precisely controlled and monitored.

#### First Principles Solution

Branching Strategy: Use a single main branch for trunk-based development.
Feature Flags: Implement these in your codebase using a configuration management system or environment variables.
Continuous Integration/Continuous Deployment (CI/CD): Integrate CI/CD pipelines to automate testing and deployment processes.
Metrics Tracking: Define key performance indicators (KPIs) such as migration lead time, rollback rate, and drift incidents.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

Migration Lead Time: Measure the average time from when a change is initiated to when it's fully deployed.
Rollback Rate: Track how often rollbacks are necessary due to issues or bugs introduced by new features.
Drift Incidents: Monitor instances where the database schema deviates from the baseline, indicating potential issues.

To achieve these goals, we will:

Use GitFlow for version control with a focus on merging changes into main frequently.
Implement feature flags via environment-specific configurations.
Utilize CI/CD tools to automate testing and deployment.
Regularly review metrics to identify bottlenecks or areas for improvement.

**Comparison:**

```
Metric	Description
Migration Lead Time	Average time from when a change is initiated to deployment
Rollback Rate	Frequency of rollbacks due to issues or bugs
Drift Incidents	Number of times the schema deviates from baseline
```

**Code:**

```python
# Example of setting up feature flags in Python using an env var
def is_feature_enabled(feature_name):
    if os.getenv(f"FEATURE_{feature_name.upper()}") == "true":
        return True
    return False

if is_feature_enabled("schema_migrations"):
    # Apply schema migrations here
```

**Example input / output:**

```text
Input: A new database schema change request is initiated.
Output:
1. Developer checks out a feature branch from main.
2. Feature branch is developed, tested, and merged into main after passing CI/CD pipeline.
3. Feature flags are toggled in the production environment to enable the new feature.
4. Metrics are logged for migration lead time, rollback rate, and drift incidents.
```

**Data:**

```yaml
# Example of a CI/CD pipeline YAML configuration snippet
stages:
  - stage: Build
    jobs:
      - job: BuildAndTest
        steps:
          - script: echo "Building and testing code"
  - stage: Deploy
    jobs:
      - job: DeployToStaging
        steps:
          - script: echo "Deploying to staging environment"
```

**Flow:**

1. Initiate Change Request
2. Checkout Feature Branch
3. Test and Develop
4. Merge into Main via CI/CDPipeline
5. Toggle Feature Flag in ProdEnv
6. Log Metrics (Migration LeadTime, Rollback Rate, DriftIncidents)

