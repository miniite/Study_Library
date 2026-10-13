---
type: "practice"
difficulty: "Deep"
---

# How would you deploy a coordinated change where a source system's event schema and your consuming pipeline both need to change together, given they live in separate repos and pipelines?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Schema Evolution in Event Driven Deploys](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/Schema_Evolution_in_Event_Driven_Deploys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To deploy a coordinated change where a source system’s event schema and a consuming data pipeline both need to change together, we must ensure that these changes are synchronized effectively across multiple repositories and pipelines. This process is critical to maintain data integrity and consistency, especially when dealing with complex schemas and intricate workflows.

#### Triage

The question asks about deploying coordinated changes in CI/CD scenarios where the event schema of a source system and a consuming pipeline need to be updated together. The key aspects are:

Repository Management: How the codebases for both systems are managed.
CI/CD Pipelines: Ensuring that changes propagate correctly across both repositories without causing data issues or delays.
Synchronization Mechanisms: Implementing a mechanism to ensure both schemas are updated in tandem.

#### First-Principles Solution
Codebase Structure:
The source system's schema is stored in one repository, and the consuming pipeline’s logic lives in another repository.
Dependency Management:
Both repositories should be set up with dependencies to each other (e.g., a library or shared package).
CI/CD Integration:
Use CI/CD tools like GitLab CI, Jenkins, GitHub Actions, etc., to trigger pipeline runs based on changes in both repositories.
Version Control:
Implement versioning for the schema and ensure that updates are tracked properly.
Change Propagation:
Ensure that any change in one system is propagated seamlessly to the other through controlled, automated processes.
Detailed Steps
Source System Schema Update:
The source system’s schema changes should be committed and pushed to its repository.
Dependency Update Notification:
A notification mechanism (e.g., GitHub webhooks) triggers a CI/CD pipeline in the consuming repository.
Pipeline Triggered:
The CI/CD pipeline in the consuming repository is triggered, ensuring that it picks up the new schema dependency and updates its logic accordingly.
Schema Validation:
Post-deployment, validate the changes to ensure the updated schemas are correctly applied without breaking existing workflows.

#### Implementation

Repository Setup:

PLAINTEXT
SourceSystem/
  ├── event-schema/
  └── other-files/

PipelineSystem/
  ├── pipeline-logic/
  └── shared-dependencies/

Dependencies and Webhooks:

The shared-dependencies directory contains a package that includes the latest schema version.
A webhook in the source system’s repository is configured to notify the CI/CD server whenever changes are pushed.

CI/CD Pipeline Configuration:

YAML
# pipeline.yml (SourceSystem)
stages:
  - deploy

variables:
  REPO_URL: "git@github.com:user/source-system.git"
  BRANCH_NAME: "main"

pipeline:
  source-schema-change:
    stage: deploy
    script:
      - git clone $REPO_URL /path/to/shared-dependencies
      - cd shared-dependencies && npm install && npm run build
      - cp event-schema/* /path/to/pipeline-logic/

# pipeline.yml (PipelineSystem)
stages:
  - deploy

variables:
  REPO_URL: "git@github.com:user/pipeline-system.git"
  BRANCH_NAME: "main"

pipeline:

#### cons
ume-schema-change:
    stage: deploy
    trigger:
      project: user/source-system
      branch: $BRANCH_NAME

#### Closing Summary
and Common Mistakes
Mistake 1: Failing to set up proper dependency management between the two repositories.
Mistake 2: Not configuring webhooks or CI/CD triggers effectively, leading to missed schema updates.
Mistake 3: Ignoring validation steps post-deployment, which can lead to runtime issues.

By following these steps and ensuring a well-coordinated approach, you can successfully deploy changes across multiple repositories while maintaining data integrity and consistency. The key is in establishing clear dependencies and automated pipelines that handle schema updates seamlessly.

**Comparison:**

| Step | Action |
|------|--------|
| 1    | Push event-schema changes to SourceSystem repository. |
| 2    | Trigger a webhook in the SourceSystem repository. |
| 3    | CI/CD pipeline in PipelineSystem is triggered. |
| 4    | PipelineSystem updates its shared dependencies with the new schema. |
| 5    | Validate and deploy the updated logic. |

**Code:**

```
pipeline.yml (SourceSystem)

stages:

deploy

variables: REPO_URL: "git@github.com:user/source-system.git" BRANCH_NAME: "main"

pipeline: source-schema-change: stage: deploy script: - git clone $REPO_URL /path/to/shared-dependencies - cd shared-dependencies && npm install && npm run build - cp event-schema/* /path/to/pipeline-logic/

CODE
Copy

```yaml
# pipeline.yml (PipelineSystem)
stages:
  - deploy

variables:
  REPO_URL: "git@github.com:user/pipeline-system.git"
  BRANCH_NAME: "main"

pipeline:
  consume-schema-change:
    stage: deploy
    trigger:
      project: user/source-system
      branch: $BRANCH_NAME
```

**Example input / output:**

```text
Input:
1. Push event-schema changes to SourceSystem.
2. Trigger a webhook in the SourceSystem repository.

Output:
1. CI/CD pipeline in PipelineSystem is triggered.
2. PipelineSystem updates its shared dependencies with the new schema.
3. Validate and deploy the updated logic.
```

**Data:**

```plaintext
SourceSystem/
├── event-schema/
│   ├── schema-v1.json
│   └── schema-v2.json
└── other-files/

PipelineSystem/
├── pipeline-logic/
│   ├── logic.js
│   └── shared-dependencies/
└── shared-dependencies/
    ├── package.json
    └── index.js
```

**Flow:**

1. Source System SchemaChange
2. Webhook Triggered
3. CI/CD Pipeline in PipelineSystem Triggers
4. Pipeline Updates SharedDependencies
5. Validation and Deployment

