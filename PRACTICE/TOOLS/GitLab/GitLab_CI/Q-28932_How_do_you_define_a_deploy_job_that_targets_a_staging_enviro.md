---
type: "practice"
difficulty: "Intermediate"
---

# How do you define a deploy job that targets a 'staging' environment?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Job Fundamentals](<../../../../TOOLS/GitLab/GitLab_CI/Job_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In this scenario, we are tasked with defining a GitLab CI pipeline that specifically targets a 'staging' environment. The goal is to ensure that code deployed to this environment undergoes automated testing and validation before it reaches production.

#### Triaging the Question

The key aspects of the question revolve around understanding how to configure a specific job in a GitLab CI pipeline. Specifically, we need to define a deploy job that runs only when targeting the 'staging' environment. This involves setting up variables, conditions, and possibly stages in our .gitlab-ci.yml file.

#### Assumptions
We are using GitLab CI/CD.
The project structure includes a standard .gitlab-ci.yml file.
Environment-specific configurations are maintained through job definitions or variables.

#### First Principles Solution
Step 1: Define the Staging Environment

First, we need to ensure that our pipeline knows about the 'staging' environment. This can be done by setting up an environment variable in GitLab CI. Typically, this is achieved under the project’s settings or within the .gitlab-ci.yml file.

YAML
# .env
STAGING_ENV: "true"
Step 2: Define the Deploy Job

Next, we define a job named deploy_staging. This job will only run when the pipeline is triggered in the 'staging' environment. We can use GitLab CI variables to conditionally execute jobs.

YAML
# .gitlab-ci.yml
stages:
  - build
  - test
  - deploy

variables:
  STAGING_ENV: "true"

deploy_staging:
  stage: deploy
  only:
    environments:
      - staging
  script:
    - echo "Deploying to Staging Environment"
    # Add your deployment steps here, e.g., deploying code to a specific server or environment.
Step 3: Verify the Pipeline Configuration

To ensure our configuration works as expected, we can run the pipeline and check its behavior in different environments. This involves committing changes and pushing them to GitLab to trigger the CI/CD pipeline.

#### Closing Summary

In this solution, we have defined a deploy job named deploy_staging that runs only when targeting the 'staging' environment. By using GitLab CI variables and conditions, we can ensure that deployments are properly managed and controlled through our pipeline configuration.

#### Common Mistakes/Follow-Ups
Incorrect Variable Usage: Forgetting to define or reference the STAGING_ENV variable correctly.
Misconfigured Stages/Phases: Not setting up the correct stages for different parts of your CI/CD process.
Overly Complex Conditions: Overcomplicating conditions can make it hard to maintain and debug.

To avoid these issues, always review and test your pipeline configuration thoroughly before moving on to production-like environments.

**Code:**

```yaml
stages:
  - build
  - test
  - deploy

variables:
  STAGING_ENV: "true"

deploy_staging:
  stage: deploy
  only:
    environments:
      - staging
  script:
    - echo "Deploying to Staging Environment"
```

