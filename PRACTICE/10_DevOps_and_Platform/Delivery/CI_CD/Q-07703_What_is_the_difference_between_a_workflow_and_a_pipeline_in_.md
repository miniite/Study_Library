---
type: "practice"
difficulty: "Basic"
---

# What is the difference between a workflow and a pipeline in tools like GitHub Actions?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Pipeline as Code](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Pipeline_as_Code.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In CI/CD tools like GitHub Actions, workflows and pipelines are often used interchangeably, but they serve different purposes. A workflow is essentially a high-level description of tasks or steps that need to be executed in a certain order. It acts as a blueprint or template for defining multiple jobs or pipelines. On the other hand, a pipeline refers to the specific execution instance of a workflow with actual input values and context.

#### Triaging the Question

The question is essentially asking you to distinguish between the concepts of workflows and pipelines in the context of CI/CD tools like GitHub Actions. This involves understanding how these terms are used within the tool and their roles in defining and executing automated builds, tests, and deployments.

#### Assumptions
You have some familiarity with basic CI/CD practices.
The focus is on explaining the difference between workflows and pipelines specifically in GitHub Actions.

#### First-Principles Solution
Workflows:

A workflow in GitHub Actions is like a script or a series of instructions that define the sequence of tasks to be executed. It acts as a template that can be reused with different inputs (e.g., branches, pull requests). A single workflow file might contain multiple jobs, which are individual steps within a job.

Pipelines:

A pipeline, in contrast, is an instance of a workflow where actual input values and context are provided. When a code change triggers the workflow, it creates one or more pipelines based on the rules defined in the workflow file. Each pipeline runs through the sequence of jobs specified in the workflow with the specific inputs.

#### Closing Summary

In summary, workflows define what tasks should be executed and how they should be structured, while pipelines are the actual executions of these workflows under specific conditions.

#### Common Mistakes/Follow-Ups
Mistake: Confusing the two terms. Both workflows and pipelines are crucial in CI/CD but serve different purposes.

#### Follow-Up
Discussing examples of when to use a single workflow with multiple pipelines or multiple workflows with a single pipeline.
Additional Context: Mentioning how these concepts apply to other CI/CD tools like Jenkins, CircleCI, etc.

**Code:**

```yaml
# Example Workflow in GitHub Actions
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
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '14'

      - name: Install dependencies
        run: npm install

      - name: Run tests
        run: npm test
```

