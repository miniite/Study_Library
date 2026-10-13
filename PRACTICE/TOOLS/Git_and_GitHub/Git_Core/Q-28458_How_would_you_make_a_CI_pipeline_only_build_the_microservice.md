---
type: "practice"
difficulty: "Intermediate"
---

# How would you make a CI pipeline only build the microservice whose files changed in a monorepo?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Monorepo CI Optimization](<../../../../TOOLS/Git_and_GitHub/Git_Core/Monorepo_CI_Optimization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To create a CI pipeline that builds only the microservice whose files have changed in a monorepo, we need to focus on leveraging Git hooks and GitHub Actions effectively. The core idea is to detect which specific microservices have been modified using Git commands and then trigger the appropriate build steps based on this information.

#### Triaging the Question

The question asks for a CI pipeline that builds only the affected microservice in a monorepo. This involves understanding:

How to identify changed files in Git.
How to configure GitHub Actions or Jenkins to respond to these changes.
How to structure the monorepo and its build processes.

#### First-Principles Solution
Step 1: Identify Changed Files

We can use git diff to determine which microservices have been modified. For a monorepo, we might tag different services with directories or specific file patterns. We need to filter out the changes to only those relevant directories.

Step 2: Configure GitHub Actions

GitHub Actions can be configured using YAML files to run workflows based on events like pushes to branches or tags. We'll set up a workflow that listens for these events and then runs the appropriate build steps.

Step 3: Execute Specific Build Steps

Based on the identified changed files, we will execute the build steps only for the affected microservices. This might involve triggering a Docker build or running tests specific to those services.

Detailed Solution

Setup Directory Structure: Suppose our monorepo has a directory structure like this:

CODE
/microservice-a
/microservice-b
...

Create Git Hooks (Optional): We can use pre-commit hooks to ensure that developers run relevant checks before committing changes. For example, if they modify microservice-a, the hook runs a specific test suite for it.

GitHub Actions Workflow: Here is a sample GitHub Actions workflow (azure-pipelines.yml or github-workflow.yml) that listens for pushes and triggers builds only for modified microservices:

YAML
name: Build Microservices

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Identify changed files
        run: |
          git diff --name-only HEAD~1 | grep -E '/microservice-[a-z]+/' > modified_files.txt

      - name: Read modified files
        id: modified-files
        run: echo "::set-output name=modified::$(cat modified_files.txt)"

      - name: Build specific microservices
        if: github.event.inputs.modified != ''
        run: |
          for file in $(cat ${{ steps.modified-files.outputs.modified }}); do
            ./build.sh ${file}
          done

#### Closing Summary
and Common Mistakes/Follow-Ups

This solution involves detecting changes, configuring a workflow to respond to these changes, and executing build processes only for the affected microservices. Common mistakes include not properly identifying changed files or failing to configure workflows accurately.

For follow-ups, one could discuss integration with other tools (like Jenkins), handling large monorepos, and ensuring robust error handling during builds.

**Code:**

```yaml
name: Build Microservices

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Identify changed files
        run: |
          git diff --name-only HEAD~1 | grep -E '/microservice-[a-z]+/' > modified_files.txt

      - name: Read modified files
        id: modified-files
        run: echo "::set-output name=modified::$(cat modified_files.txt)"

      - name: Build specific microservices
        if: github.event.inputs.modified != ''
        run: |
          for file in $(cat ${{ steps.modified-files.outputs.modified }}); do
            ./build.sh ${file}
          done
```

**Flow:**

1. Yes
2. No
3. Checkout Repository
4. Identify Changed Files
5. Modified Microservices?
6. Build Specific Microservices
7. Skip Build Steps

