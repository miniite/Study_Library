---
type: "practice"
difficulty: "Basic"
---

# How do you make one job wait for another job to finish before it starts?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Workflow Fundamentals](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Workflow_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a GitHub Actions job waits for another job to complete before starting, we can leverage the needs keyword and concurrency settings in our workflows. Essentially, this is about managing dependencies between jobs so they execute in the correct order.

First, let's understand what we're dealing with. In GitHub Actions, each step within a job is independent by default, but you might want to wait for one job to finish before another starts. This could be due to dependency on the output of one job or ensuring that all necessary setup is completed first.

Here are the steps and reasoning involved:

Define the Jobs: Each job in your workflow file (.yml) can have dependencies defined using needs.
Use Needs Keyword: By specifying a job name under the needs keyword, you ensure that only after the specified jobs complete will the current job start.
Concurrency Control: Use concurrency to control how many instances of a job are allowed to run at once.

Let's consider an example where we have two jobs: build and deploy. The deploy job should only start after the build job has completed successfully.

Here is how you can define this in your GitHub Actions workflow file:

YAML
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Build project
        run: |
          # Your build commands here

  deploy:
    needs: build
    runs-on: ubuntu-latest
    concurrency:
      group: deployment
      cancel-in-progress: true
    steps:
      - name: Deploy to production
        run: |
          # Your deployment commands here

In this example, the deploy job waits for the build job to complete before it starts. The concurrency section ensures that only one instance of the deploy job runs at a time across all branches, and if another deploy is triggered while an existing one is in progress, the new request will be canceled.

**Code:**

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Build project
        run: |
          # Your build commands here

  deploy:
    needs: build
    runs-on: ubuntu-latest
    concurrency:
      group: deployment
      cancel-in-progress: true
    steps:
      - name: Deploy to production
        run: |
          # Your deployment commands here
```

