---
type: "practice"
difficulty: "Intermediate"
---

# Why are local Git hooks not shared across a team by default, and how does the pre-commit framework solve that?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Pre-commit Hooks](<../../../../TOOLS/Git_and_GitHub/Git_Core/Pre-commit_Hooks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Local Git hooks are essential tools in a developer's workflow, but they are not shared by default due to their highly specific nature and potential impact on commit quality. The pre-commit framework addresses this by providing a structured way to share common pre-commit checks across a team, ensuring consistent code quality without interfering with local workflows.

The key idea behind why local Git hooks aren’t shared is that they are designed for individual or small-scale projects. They can include file manipulation, custom validation scripts, and other actions specific to the project's needs. Sharing these globally could lead to conflicts or unexpected behavior across different repositories and environments. The pre-commit framework solves this by allowing teams to define a standard set of hooks in a shared repository that are then automatically installed when developers initialize their local projects.

To illustrate, let’s break down how pre-commit works from first principles:

Hook Definition: Developers define the pre-commit hooks they need for their project.
Configuration File: These hooks are configured in a .pre-commit-config.yaml file within a dedicated repository or in each developer's local environment.
Automatic Installation: When developers use pre-commit to install hooks, it ensures that these hooks are correctly set up and available locally without needing to be shared directly with other repositories.

This approach maintains the flexibility of custom hooks while enabling collaboration and consistency across teams.

**Code:**

```yaml
# Example .pre-commit-config.yaml file
repos:
- repo: https://github.com/pre-commit/pre-commit-hooks
  rev: v4.3.0
  hooks:
  - id: trailing-whitespace
  - id: end-of-file-fixer

- repo: https://github.com/psf/black
  rev: stable
  hooks:
  - id: black
```

**Example input / output:**

```text
No specific input-output examples are needed for this concept, but here is a typical workflow:

1. Developer clones the `.pre-commit-config.yaml` file from a shared repository.
2. They run `pre-commit install`, which installs the specified hooks locally.
3. When they commit changes, `pre-commit` runs all configured hooks before the actual Git commit.
```

**Flow:**

1. Yes
2. Developer Initializes LocalEnvironment
3. Install Pre-Commit Hooks?
4. Run pre-commit install
5. Clone .pre-commit-config.yamlfrom Shared Repository
6. Locally Install ConfiguredHooks
7. Commit Code with pre-commitRunning Checks

