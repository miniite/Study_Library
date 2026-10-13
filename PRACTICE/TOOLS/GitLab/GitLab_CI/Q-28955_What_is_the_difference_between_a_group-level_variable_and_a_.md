---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between a group-level variable and a project-level variable in GitLab?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Cache vs Artifacts](<../../../../TOOLS/GitLab/GitLab_CI/Cache_vs_Artifacts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Group-level variables and project-level variables are both used to store configuration data within GitLab, but they serve different purposes and have distinct scopes. This answer will explain their differences from first principles, starting with a brief overview of each type and then delving into how they differ.

Firstly, group-level variables can be shared across all projects that belong to a particular group in GitLab. These variables are useful for settings that should remain consistent across multiple projects within the same scope. On the other hand, project-level variables are specific to an individual project and can override any group-level settings if needed. Understanding these differences is crucial for managing configurations effectively in your development environment.

To understand the distinction between them, let's break it down step by step:

Scope: Group-level variables have a broader scope, affecting all projects within a group hierarchy. Project-level variables are confined to individual projects.
Inheritance and Override: Group-level variables can be inherited by projects within the same group. However, project-level settings can override these group-wide defaults if necessary.
Visibility: Both types of variables can be public or private. Public variables are visible to everyone in a group, while private ones are accessible only to group members with appropriate permissions.

Now let's explore some practical scenarios where understanding this distinction is critical:

Suppose you have a development group for your application and want to set up environment-specific configurations like database credentials or API keys. You would use project-level variables at the project level to manage these settings per each project.
For shared settings that should be consistent across multiple projects, such as branch protection rules or predefined pipeline stages, using group-level variables makes sense.

In summary, while both types of variables serve a similar purpose in managing configuration data within GitLab, their use cases and scopes make them distinct. The choice between the two depends on whether you need shared settings that should be consistent across projects (group-level) or specific configurations unique to each project (project-level).

**Comparison:**

```
Variable Type	Scope	Inheritance	Override
Group-level	Group	Yes	No
Project-level	Project	Optional	Yes
```

**Code:**

```yaml
# Example of group-level variable in a group settings file
variables:
  - key: GROUP_DB_PASSWORD
    value: "s3cr3t"
    protected: true

# Example of project-level variable in a project settings file
variables:
  - key: PROJECT_API_KEY
    value: "aBcDeFgH123"
    protected: true
```

