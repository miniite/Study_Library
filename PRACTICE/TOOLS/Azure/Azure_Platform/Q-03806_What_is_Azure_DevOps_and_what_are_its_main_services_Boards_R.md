---
type: "practice"
difficulty: "Basic"
---

# What is Azure DevOps and what are its main services (Boards/Repos/Pipelines/Artifacts/Test Plans)?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Azure Pipelines and Release Approvals](<../../../../TOOLS/Azure/Azure_Platform/Azure_DevOps_and_Artifact_Governance/Azure_Pipelines_and_Release_Approvals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

Azure DevOps is a comprehensive suite of tools designed to streamline software development processes across teams, integrating version control, project management, and continuous integration & delivery (CI/CD). Its main services—Boards, Repositories, Pipelines, Artifacts, and Test Plans—are essential for orchestrating the entire software lifecycle from planning to deployment.

#### Triaging the Question

The question is asking you to explain Azure DevOps and its primary services. The interviewer wants to understand your foundational knowledge of these tools and how they fit together in a software development workflow. You should clarify what each service does and their roles in managing projects effectively.

Key Assumptions:
Basic understanding of CI/CD and DevOps practices.
Familiarity with Azure as a cloud platform.

#### First Principles Solution
1. Azure DevOps Overview

Azure DevOps is a powerful tool suite that integrates multiple services to support the entire software development lifecycle, from planning and coding to testing and deployment. It is built on Microsoft’s cloud infrastructure, offering robust features for developers, project managers, and other stakeholders.

2. Boards
Purpose: Boards in Azure DevOps provide a visual workspace where you can manage tasks and work items.
Key Features:
Kanban boards help with agile development by allowing teams to visualize their workflow and identify bottlenecks.
Backlogs enable prioritization of features and stories, ensuring that the most critical tasks are addressed first.
3. Repos
Purpose: Repositories store source code and manage version control.
Key Features:
Git repositories support branching, merging, and tagging for collaborative coding.
Code review tools ensure that changes meet quality standards before being merged into the main branch.
4. Pipelines
Purpose: Pipelines automate the build, test, and deployment processes to streamline software release cycles.
Key Features:
CI/CD pipelines can be configured using YAML or through visual editors.
Multiple stages allow for granular control over the build and deploy process.
Integration with various tools like Docker, Kubernetes, and cloud services ensures a seamless deployment pipeline.
5. Artifacts
Purpose: Artifacts store built products and associated metadata, making them available for testing and production.
Key Features:
Built-in support for storing and managing artifacts such as binaries, packages, and Docker images.
Secure access controls to ensure that only authorized personnel can retrieve and use these artifacts.
6. Test Plans
Purpose: Test plans enable the creation of detailed test cases and execution strategies.
Key Features:
Automated testing frameworks for unit tests, integration tests, and end-to-end tests.
Quality gates to prevent deployment if certain tests fail.
Reporting tools to track test results and identify areas for improvement.

#### Closing Summary

Azure DevOps offers a holistic approach to software development by integrating multiple services that cover all aspects of the development lifecycle. Boards help with task management, Repositories handle code versioning, Pipelines automate deployments, Artifacts store built products securely, and Test Plans ensure robust quality checks.

#### Common Mistakes/Follow-Ups
For fresher/2/3: Failing to mention key features or mixing up services.
For principal/architect: Overlooking advanced configurations or not discussing scalability and integration aspects.

If asked for more details, you could discuss how these services interact with each other or explore their integrations with other Azure services like Kubernetes or Azure DevTest Labs.

**Comparison:**

| Boards | Repositories | Pipelines | Artifacts | Test Plans |
|--------|--------------|-----------|-----------|------------|
| Kanban & Backlog | Git Repositories | CI/CD Pipelines | Built Products | Detailed Test Cases |

**Code:**

```yaml
# Example CI/CD Pipeline in YAML
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

steps:
- script: echo Hello, world!
  displayName: 'Run a one-line script'

- task: PublishBuildArtifacts@1
  inputs:
    PathtoPublish: '$(Build.ArtifactStagingDirectory)'
    ArtifactName: 'drop'
    publishLocation: 'Container'
```

**Example input / output:**

```text
Input (Request to Create a New Repository):
- Project Name: MyProject
- Version Control: Git

Output (Response from Azure DevOps):
{
  "repositoryId": "R001",
  "url": "https://dev.azure.com/organization/project/_git/repo",
  "status": "created"
}
```

**Data:**

```json
{
  "boards": {
    "backlog": [
      {"id": "B001", "title": "Feature A"},
      {"id": "B002", "title": "Bug B"}
    ],
    "kanban": [
      {"id": "K001", "state": "To Do", "title": "Task 1"},
      {"id": "K002", "state": "In Progress", "title": "Task 2"}
    ]
  },
  "repos": {
    "git": [
      {"url": "https://dev.azure.com/organization/project/_git/repo"},
      {"url": "https://github.com/user/repo.git"}
    ]
  },
  "pipelines": {
    "ci": [
      {"name": "Build Pipeline", "definitionId": 1, "status": "active"}
    ],
    "cd": [
      {"name": "Deployment Pipeline", "stage": ["Dev", "Staging", "Prod"], "status": "completed"}
    ]
  },
  "artifacts": {
    "packages": [
      {"name": "myapp.zip", "url": "https://dev.azure.com/organization/project/_apis/build/builds/1/artifacts?artifactName=myapp.zip"}
    ],
    "dockerImages": [
      {"name": "myapp:latest", "url": "https://dev.azure.com/organization/project/_apis/docker/images/myapp?tag=latest"}
    ]
  },
  "testPlans": {
    "plans": [
      {"id": "T001", "title": "Integration Tests", "status": "completed"},
      {"id": "T002", "title": "Performance Tests", "status": "in-progress"}
    ]
  }
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. A[Start] --> B{Boards};
4. B --> C[Kanban Board];
5. B --> D[Backlog];
6. C --> E[Task Management];
7. D --> F[Prioritization];
8. G[Repos] --> H{Repositories};
9. H --> I[Version Control];
10. H --> J[GIT Repositories];
11. K[Pipelines] --> L{Pipelines};
12. L --> M[Build Automation];
13. L --> N[Test Automation];
14. O[Artifacts] --> P{Artifacts};
15. P --> Q[Built Products];
16. P --> R[Docker Images];
17. S[Test Plans] --> T{Test Plans};
18. T --> U[Automation];
19. T --> V[Quality Gates];
20. C -. Interacts with-.> K;
21. H -. Integrates with-.> O;
22. L -. Deploys to-.> Q;
23. T -. Monitors-.> N;

