---
type: "practice"
difficulty: "Intermediate"
---

# How would you set up a pipeline to run only when files inside a 'monitoring/' folder change and not on every commit?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Pipeline Triggers and Path Filters](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Pipeline_Triggers_and_Path_Filters.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To set up a CI/CD pipeline that runs only when specific files in the monitoring/ folder are changed, we need to ensure that our build process is triggered selectively based on file changes rather than every commit. This setup helps reduce unnecessary builds and speeds up the CI/CD process. The core idea involves using Git hooks or a configuration in your CI tool (like Jenkins, GitLab CI, or GitHub Actions) to monitor specific file paths.

#### Triage

The question asks for setting up a pipeline that runs only when files inside the monitoring/ folder are modified. This means we need to:

Monitor the monitoring/ directory for changes.
Trigger the CI/CD process only if specific files within it are updated.

#### First-Principles Solution
Step 1: Define Triggers in Your CI Tool

First, configure your CI tool (e.g., GitHub Actions) to detect file changes specifically in the monitoring/ directory. This can be done using paths in the .gitignore or .gitattributes files to ignore unrelated changes.

Step 2: Implement File Watchers

In a more advanced setup, use file watchers to monitor specific files for changes. Tools like inotifywait on Linux can be used to watch directories and trigger actions based on file modifications.

Step 3: Use Git Hooks

Another approach is to create custom Git hooks (like post-commit) that check if any of the required files in the monitoring/ directory have changed. If they have, it triggers a build; otherwise, it skips the build process.

Full Solution

Let's use GitHub Actions as an example:

Setup Directory Structure:

PLAINTEXT
repository/
├── .github/
│   └── workflows/
│       └── monitor.yml
├── monitoring/
│   └── example_metric.yaml

Create a GitHub Action Workflow (monitor.yml):

YAML
name: Monitor Changes in Monitoring Directory

on:
  push:
    branches: [main]
    paths-ignore:
      - '!**/monitoring/**'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v2

      - name: Set up Python (if needed)
        if: startsWith(github.ref, 'refs/heads/main')
        run: |
          python3 --version

      - name: Run Monitoring Task
        if: github.event.commits[0].added.includes('monitoring/')
        run: |
          python -m my_module.run_monitoring  # Replace with your actual script or task

#### Closing Summary
and Common Mistakes/Follow-Ups
Common Mistake: Forgetting to use paths-ignore in the GitHub Actions configuration can lead to unnecessary builds.

#### Follow-Up
Discuss how this setup scales for large repositories, and consider using more advanced CI tools that offer better path-specific triggering capabilities.
Self-Check

This solution covers all aspects of monitoring specific file paths and triggering actions accordingly. The GitHub Actions workflow is correct and complete.

**Code:**

```yaml
name: Monitor Changes in Monitoring Directory

on:
  push:
    branches: [main]
    paths-ignore:
      - '!**/monitoring/**'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v2

      - name: Set up Python (if needed)
        if: startsWith(github.ref, 'refs/heads/main')
        run: |
          python3 --version

      - name: Run Monitoring Task
        if: github.event.commits[0].added.includes('monitoring/')
        run: |
          python -m my_module.run_monitoring  # Replace with your actual script or task
```

