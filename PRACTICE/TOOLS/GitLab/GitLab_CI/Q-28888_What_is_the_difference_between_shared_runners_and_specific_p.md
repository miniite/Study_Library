---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between shared runners and specific (project) runners?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Job Fundamentals](<../../../../TOOLS/GitLab/GitLab_CI/Job_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Shared runners and specific (project) runners in GitLab CI are two different ways to manage your Continuous Integration/Continuous Deployment (CI/CD) pipelines, each with its own use cases and benefits. In a nutshell, shared runners can be used by multiple projects, whereas specific (or project) runners are dedicated to individual projects.

To tackle this question from first principles, we'll start by understanding what GitLab CI is, the roles of runners in the pipeline process, and then dive into the specifics of each runner type.

**Comparison:**

```
Runner Type	Description	Use Case
Shared	Can be used by multiple projects.	Cost-effective for shared tasks.
Project	Dedicated to a single project.	Ensures isolation and security.
```

**Code:**

```
MERMAID
Copy
graph TB
    Runner{Runners}
    SharedRunner[Shared Runners]
    ProjectRunner[Project-specific Runners]
    Pipeline[Pipeline Execution]

    Runner -->|Allocated based on need| Pipeline
    SharedRunner -->|Allocated based on project| Pipeline
    ProjectRunner -->|Alone for the project| Pipeline
```

**Example input / output:**

```

An example of a pipeline configuration might look like this:

YAML
Copy
stages:
  - build
  - test

shared_runner_job:
  stage: build
  script:
    - echo "Building using shared runner"

project_runner_job:
  stage: test
  script:
    - echo "Testing with project-specific runner"
```

**Data:**

```json
{
  "runner": {
    "type": "runner_type",
    "status": "active"
  },
  "project_runner": {
    "type": "specific_runner",
    "projectId": "123456789"
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Shared Runner
6. Is the job available?
7. Allocate runner
8. Retry or fail pipeline
9. Project-specific Runner
10. Is the project configured?
11. Allocate runner
12. Retry or fail pipeline

