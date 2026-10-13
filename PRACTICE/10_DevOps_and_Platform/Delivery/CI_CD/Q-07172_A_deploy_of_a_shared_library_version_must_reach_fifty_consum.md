---
type: "practice"
difficulty: "Deep"
---

# A deploy of a shared library version must reach fifty consumer services, but you want the library team accountable for breakage, not the consumers; how do you architect the responsibility and the gating so incentives align?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Shared Pipeline Changes](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're aiming to ensure that when a shared library is deployed, it successfully reaches all consumer services, while keeping accountability on the library team. The key challenge here is to design a robust CI/CD pipeline with clear gating mechanisms and metrics that align incentives.

The core idea is to implement a strict change control process where the library team must validate new versions across multiple stages before they are allowed into production. This ensures that any issues discovered can be addressed by the original developers, maintaining their responsibility for breakage.

**Comparison:**

| Stage         | Description                                                                                           |
|---------------|-------------------------------------------------------------------------------------------------------|
| Pre-Commit    | Code reviews and automated tests                                                                      |
| Review & Test | Additional manual testing by a review team                                                             |
| Beta Rollout  | Rolling out to beta services for initial testing                                                      |
| Staging       | Full staging environment integration, monitoring for issues                                           |
| Production    | Deployment to all consumer services                                                                   |

**Code:**

```
MERMAID
Copy
graph TD;
    A[Library Change Request] --> B(Pre-Commit);
    B --> C(Review & Test);
    C --> D(Beta Rollout);
    D --> E(Staging);
    E --> F(Production) [Deployment];
```

**Example input / output:**

```

An example input could be a request to deploy version 1.2 of the library:

JSON
Copy
{
  "library_version": "1.2",
  "requested_by": "Library Team A",
  "status": "Pre-Commit"
}

After successful pre-commit checks, it might transition through stages like this:

JSON
Copy
[
  {"stage": "Review & Test", "status": "Passed"},
  {"stage": "Beta Rollout", "status": "Passed"},
  {"stage": "Staging", "status": "Passed"},
  {"stage": "Production", "status": "Deployed"}
]
```

**Data:**

```yaml
pipeline_stages:
  - name: Pre-Commit
    description: Code reviews and automated tests on the library
  - name: Review & Test
    description: Additional manual testing by a review team
  - name: Beta Rollout
    description: Rolling out to beta services for initial testing
  - name: Staging
    description: Full staging environment integration, monitoring for issues
  - name: Production
    description: Deployment to all consumer services
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph LR;
3. A[Library Team] --> B(Submit Change Request);
4. B --> C(Pull Requests + Automated Tests) [Pre-Commit];
5. C --> D(Review & Test);
6. D --> E(Beta Rollout);
7. E --> F(Staging);
8. F --> G(Production) [Deploy to All Services];

