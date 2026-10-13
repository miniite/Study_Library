---
type: "practice"
difficulty: "Basic"
---

# What is a pipeline in GitLab CI and what triggers it to run?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Job Fundamentals](<../../../../TOOLS/GitLab/GitLab_CI/Job_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In GitLab CI, a pipeline is an automated process that builds, tests, and deploys your application based on the changes committed to the repository. It's essentially a series of jobs orchestrated in stages that run whenever specified events occur.

To understand what triggers a pipeline, first, let’s break down its components:

Pipelines: These are defined in .gitlab-ci.yml files within your project.
Jobs: Each job is a task in the pipeline (e.g., building an image, running tests).
Stages: Jobs are grouped into stages to represent different phases of the deployment process.
Triggers and Events

Now, let’s look at what triggers these pipelines. A GitLab CI pipeline can be triggered by several events:

Pushes to Repository:
When you push changes to any branch or tag in your repository.
Scheduled Jobs:
You can set up cron jobs to run specific tasks at regular intervals.
Merge Requests:
Pipelines can be triggered when a merge request is created, updated, or merged into a target branch.
Tagging Events:
When you tag your repository with a specific version.

#### Example
.gitlab-ci.yml File

Here’s an example configuration that demonstrates how to define and trigger jobs in a pipeline:

YAML
stages:
  - build
  - test
  - deploy

build_job:
  stage: build
  script:
    - echo "Building the image..."
    - docker build -t my-image .

test_job:
  stage: test
  script:
    - echo "Running tests..."
    - pytest

deploy_job:
  stage: deploy
  only:
    - master
  script:
    - echo "Deploying to production..."
Understanding the Example
Stages: The pipeline is split into three stages—build, test, and deploy.
Jobs: Each job corresponds to a step within its stage.
Triggers (only:): The deploy_job will only run when changes are pushed to the master branch.
Common Scenarios
Push Trigger:
Every time you push code, GitLab CI checks if there is a .gitlab-ci.yml file and runs the defined jobs.
Merge Request Trigger:
A developer creates or updates a merge request to integrate changes into master. This action can trigger a pipeline that includes testing and deployment steps.

#### Common Mistakes
and Follow-Ups
Misunderstanding Triggers: Developers often miss specifying when a job should run, leading to unnecessary executions. Ensure you correctly configure the only: or when: keywords.
Ignoring Environment-Specific Jobs: Failing to include environment-specific triggers (e.g., only deploying to production on certain branches) can lead to security and compliance issues.

In summary, a GitLab CI pipeline is triggered by events like pushes, merge requests, or scheduled tasks. Each job within the pipeline executes based on its configuration in the .gitlab-ci.yml file.

**Code:**

```yaml
stages:
  - build
  - test
  - deploy

build_job:
  stage: build
  script:
    - echo "Building the image..."
    - docker build -t my-image .

test_job:
  stage: test
  script:
    - echo "Running tests..."
    - pytest

deploy_job:
  stage: deploy
  only:
    - master
  script:
    - echo "Deploying to production..."
```

