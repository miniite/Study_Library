---
type: "practice"
difficulty: "Basic"
---

# What is the purpose of the .github/workflows directory and what file format do workflows use?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Self-hosted Runners](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Self-hosted_Runners.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

The .github/workflows directory in a GitHub repository is where you define your CI/CD (Continuous Integration/Continuous Deployment) pipelines using YAML files. These workflows automate various tasks, such as running tests and building software, right from the source code repository.

#### Triaging the Question

The question asks two parts: first, about the purpose of the .github/workflows directory; second, about the file format used by these workflows. The core idea here is to understand how GitHub Actions are set up and configured within a project.

Detailed Explanation:
Purpose of the .github/workflows Directory

GitHub Actions are automation tools provided by GitHub that you can use to automate your development workflow. When you create an action, it's stored in the .github/workflows directory within your repository. This directory acts as a central hub where all your CI/CD configurations live.

GitHub recommends placing your workflows in this specific directory because it is recognized and automatically picked up by GitHub Actions. By convention, files inside .github/workflows are YAML configuration files that define the actions you want to run at various stages of your project lifecycle (e.g., on push events, pull requests, or schedule-based triggers).

File Format: YAML

The file format used for defining workflows is YAML (YAML Ain't Markup Language). YAML provides a human-readable data serialization standard that makes it easy to define complex structures and nested configurations. Here's an example of what a simple workflow might look like:

YAML
# Example GitHub Action Workflow
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '16'
      - run: npm ci
      - run: npm test
name: This is the name of your workflow.
on: Specifies the event that triggers this workflow. In this example, it’s triggered on push to the main branch and pull_request events.
jobs: Defines a job that will run as part of the workflow. Here, the job is named "build".
runs-on: Specifies where the job should be executed (e.g., an Ubuntu VM).
steps: A list of steps to perform within this job.

#### Common Mistakes
and Follow-Ups:
Misunderstanding the .github/workflows Directory: Sometimes, developers might confuse this with other directories in GitHub repositories. It’s important to understand that it's a specific place designated for defining workflows.
Incorrect YAML Syntax: YAML is sensitive to indentation and formatting. A single misplaced space can cause your workflow to fail.

When designing and implementing CI/CD pipelines using GitHub Actions, ensure you follow best practices such as versioning your workflows, documenting them, and testing them thoroughly.

#### Closing Summary

In summary, the .github/workflows directory serves as a centralized location for defining automated tasks in your GitHub repository. These tasks are written in YAML files to create CI/CD pipelines that help streamline development processes.

**Code:**

```yaml
# Example GitHub Action Workflow
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '16'
      - run: npm ci
      - run: npm test
```

