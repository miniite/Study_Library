---
type: "practice"
difficulty: "Basic"
---

# How do you make one CI job wait for another to finish using dependencies, for example build then deploy?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Reducing Actions Costs](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reducing_Actions_Costs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a CI (Continuous Integration) job waits for another job to complete before proceeding, we can use dependencies in GitHub Actions. The key idea is to structure our workflow YAML file so that one job depends on the successful completion of another. This ensures that the build step completes before initiating the deployment step.

#### Triage
The question asks how to make a CI job wait for another using dependencies. In this context, we're talking about ensuring that the "build" job runs and completes successfully before starting the "deploy" job in GitHub Actions workflows.

#### Assumptions

We are working with GitHub Actions.
We have two jobs: one for building (e.g., compiling code) and one for deploying (e.g., pushing to a production environment).
The build must complete successfully before deployment can start.

#### First-principles solution

Define the "build" job in your workflow YAML file:
YAML
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Run build script
        run: make
Define the "deploy" job and specify that it depends on the success of the "build" job:
YAML
  deploy:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy application
        run: echo "Deploying..."

In this setup, the needs keyword ensures that the "deploy" job will only start after the "build" job has successfully completed. If the build fails, the deployment will not proceed.

#### Closing summary
and common mistakes/follow-ups: The key to ensuring one CI job waits for another is using dependencies within GitHub Actions workflows via the needs keyword. Common mistakes include forgetting to define the dependency or mistakenly placing conditions that prevent successful execution of subsequent jobs. Follow-up questions might involve discussing how failures are handled, what happens if a build fails, and whether retries should be implemented.

