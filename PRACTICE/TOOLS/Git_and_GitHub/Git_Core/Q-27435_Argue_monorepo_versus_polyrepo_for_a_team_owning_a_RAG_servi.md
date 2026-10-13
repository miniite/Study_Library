---
type: "practice"
difficulty: "Deep"
---

# Argue monorepo versus polyrepo for a team owning a RAG service, an agent service, and an eval platform, specifically around atomic cross-service prompt changes, CI blast radius, and per-service access control.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Monorepo CI Optimization](<../../../../TOOLS/Git_and_GitHub/Git_Core/Monorepo_CI_Optimization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to decide between using a monorepo or polyrepo structure for our team's repositories, focusing on three key aspects: atomic cross-service prompt changes, CI blast radius, and per-service access control. Let's break down each aspect from first principles.

#### Triage

The question is essentially about balancing the benefits of code organization versus the complexity it introduces in a multi-repository setup. We are probing how well you can articulate trade-offs related to version control, collaboration, and security.

Atomic Cross-Service Prompt Changes

In a monorepo, all services (RAG service, agent service, eval platform) share a single repository, making cross-service prompt changes atomic and easier to manage. This is because the entire codebase is in one place, allowing for seamless integration of new features or bug fixes across all services without context switching.

CI Blast Radius

Polyrepo can minimize the blast radius by isolating changes within individual repositories. If a change breaks a service, it’s contained within that repo and less likely to impact other services. However, this also means that cross-service integrations require more complex setup and coordination among multiple CI pipelines.

Per-Service Access Control

Monorepo can offer simpler access control because developers only need permissions for the monorepo as a whole. In polyrepo, finer-grained controls are necessary to restrict access to specific parts of each service, which can be administratively cumbersome.

Step-by-Step Solution
Monorepo Approach
Atomicity and Simplicity: All services share a single repository, making cross-service operations atomic.
Centralized CI/CD: A unified CI pipeline can handle all services efficiently, reducing the risk of breaking changes affecting multiple services simultaneously.
Access Control: Simplified access management since developers only need permissions for one repo.
Polyrepo Approach
Blast Radius Minimization: Changes are isolated within individual repos, reducing the risk of cascading failures.
Scalable CI Pipelines: Each service can have its own CI pipeline tailored to specific needs without affecting others.
Fine-Grained Access Control: Better control over who can access different parts of each service.

#### Closing Summary

For our team's RAG, agent, and eval services, a monorepo approach appears more suitable due to the benefits in atomic cross-service prompt changes, centralized CI/CD processes, and simpler overall access control. However, if the services are highly independent or have significantly different release cycles, a polyrepo setup could be beneficial by minimizing blast radius and allowing for more granular CI configurations.

#### Common Mistakes/Follow-Ups
Overlooking the importance of atomicity in cross-service changes.
Ignoring the administrative overhead of fine-grained access control in polyrepos.
Underestimating the complexity of maintaining multiple CI pipelines.

**Comparison:**

| Aspect           | Monorepo                     | Polyrepo                    |
|------------------|------------------------------|----------------------------|
| Atomicity        | High                         | Lower                      |
| CI Blast Radius  | Large                        | Small                      |
| Access Control   | Simple                       | Complex                    |

**Code:**

```python
# Example of a simple function to illustrate atomic operations
def apply_prompt_change(service_code):
    service_code.apply_new_prompt()  # Atomic change within monorepo
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Apply prompt change in RAG service and agent service simultaneously (monorepo) | Successfully applied changes to both services without conflicts. |
| Apply prompt change only in RAG service using polyrepo approach | Only the RAG service CI pipeline triggered, minimizing blast radius. |

**Data:**

```json
{
  "repo_type": "monorepo",
  "services": [
    {
      "name": "RAG Service",
      "dependencies": ["Agent Service", "Eval Platform"]
    },
    {
      "name": "Agent Service",
      "dependencies": ["RAG Service", "Eval Platform"]
    },
    {
      "name": "Eval Platform",
      "dependencies": ["RAG Service", "Agent Service"]
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Monorepo
6. Is cross-service atomic?
7. Centralized CI/CD
8. Fine-grained access control
9. Polyrepo
10. Is service highly independent?
11. Scalable CI Pipelines
12. Minimized blast radius

