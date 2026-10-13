---
type: "practice"
difficulty: "Deep"
---

# Explain the difference between include:project with a ref and CI/CD component versions for shared pipeline logic.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Cache vs Artifacts](<../../../../TOOLS/GitLab/GitLab_CI/Cache_vs_Artifacts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To explain the differences between include:project with a ref versus using CI/CD component versions for shared pipeline logic in GitLab CI, let's start by understanding what these terms mean and their implications.

In GitLab CI, include:project allows you to reuse pipeline definitions from other projects or specific branches (using the ref parameter). This is useful when you have common pipeline steps that are shared across multiple projects. On the other hand, using CI/CD component versions involves managing and versioning reusable components separately, which can be more modular and easier to manage in large organizations.

First, let's delve into the specifics of each approach:

#### Triage
What is being Probed

The interviewer is likely probing your understanding of GitLab CI's flexibility, how shared pipeline logic is managed, and the trade-offs between different approaches. They may also be interested in your experience with CI/CD best practices and versioning strategies.

Key Assumptions
You have a basic understanding of GitLab CI and its structure.
You are familiar with the concept of including pipelines from other projects or branches.
You understand the role of component versions in managing reusable components.
Step-by-Step Solution
include:project with a ref

Definition: When you use include:project, GitLab includes pipeline definitions from another project directly into your current pipeline file. This is useful for sharing logic across projects.

Use Case: Ideal when the shared logic is relatively stable and changes infrequently, or when you want to share common steps between projects without versioning them separately.

#### Example

YAML
include:
  - project: 'gitlab-org/gitlab-ci'
    ref: 'main'  # Use a specific branch or tag

Advantages:

Simple to implement and manage.
Reduces duplication of code across multiple projects.

Disadvantages:

Changes in the included project can affect your pipeline if it's not versioned properly.
No explicit version control over shared logic, which can lead to drift issues.
CI/CD Component Versions

Definition: Using component versions involves creating and managing reusable components that are stored separately from your main projects. Each component has its own lifecycle and versioning system.

Use Case: Best for scenarios where you need fine-grained control over shared logic, or when the shared code changes frequently and needs to be managed independently.

#### Example

YAML
include:
  project: 'gitlab-org/gitlab-ci'
  ref: 'main'  # Use a specific branch or tag

Advantages:

Clear versioning of shared logic, making it easier to track changes and dependencies.
More modular approach, allowing for better separation of concerns.

Disadvantages:

Additional complexity in managing components and their versions.
Requires more setup and maintenance compared to simple include:project.

#### Closing Summary

In summary, the choice between using include:project with a specific ref or CI/CD component versions depends on your project's requirements. The former is simpler but can lead to drift issues, while the latter offers better control and modularity at the cost of additional setup.

#### Common Mistakes
and Follow-Ups
Mistake: Ignoring versioning when using include:project can result in unexpected pipeline behavior.

#### Follow-Up
Discuss how to manage dependencies between different components or projects, especially in complex multi-project setups.
Additional Insight: Mention the importance of considering your team's size and project complexity before deciding on a strategy.

**Comparison:**

```
Approach	include:project with Ref	CI/CD Component Versions
Control	Limited	High
Complexity	Simple	Higher
Maintenance	Easier	More effort
Isolation	Less	Better
```

**Code:**

```yaml
# Example using include:project with ref
include:
  - project: 'gitlab-org/gitlab-ci'
    ref: 'main'

# Example using component versioning (hypothetical)
component:
  name: 'shared_pipeline'
  version: '1.0.0'
  pipeline_file_path: 'path/to/pipeline.yml'
```

**Example input / output:**

```text
Input:
include:
  - project: 'gitlab-org/gitlab-ci'
    ref: 'main'

Output (Dry-run):
- Pipeline steps from 'gitlab-org/gitlab-ci' project at 'main' ref are included in the current pipeline.
```

**Data:**

```yaml
# Sample data for including a project with ref
include:
  - project: 'gitlab-org/gitlab-ci'
    ref: 'main'

# Example component versioning setup (simplified)
component:
  name: 'shared_pipeline'
  version: '1.0.0'
  pipeline_file_path: 'path/to/pipeline.yml'
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Include Project with Ref
6. Is Shared Logic Stable?
7. Simple and Easy
8. Manage DependenciesCarefully
9. Component Versions
10. Version Control Needed?
11. Use Versioning
12. Simplify Management
13. Reduce Duplication
14. Handle Drift Issues
15. Clear Track of Changes
16. Manage ComponentsIndependently

