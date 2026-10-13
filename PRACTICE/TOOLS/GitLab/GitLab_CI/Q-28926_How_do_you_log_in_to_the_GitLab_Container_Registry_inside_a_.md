---
type: "practice"
difficulty: "Intermediate"
---

# How do you log in to the GitLab Container Registry inside a job using docker login?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Docker Layer Caching](<../../../../TOOLS/GitLab/GitLab_CI/Docker_Layer_Caching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To log into the GitLab Container Registry inside a job using docker login, we need to use the credentials stored in the GitLab CI/CD pipeline. This process ensures that the container image can be pushed or pulled from the registry securely. Let's break down how this is achieved step-by-step.

First, understand that you will require two pieces of information: your GitLab username and a personal access token (PAT) with read/write access to the container registry. The PAT acts as an alternative to basic authentication when pushing/pulling images via docker login.

#### Triage

The question asks how to use docker login inside a job, which means we need to configure a GitLab CI/CD pipeline where this command is executed within one of its jobs. This setup involves setting up the necessary environment variables and ensuring that the Docker daemon has access to these credentials.

Step-by-Step Solution

Create Personal Access Token:

Go to your GitLab account settings.
Navigate to "Access Tokens" under "Settings".
Create a new token with appropriate permissions (usually read_registry and write_registry).

Configure GitLab CI/CD Pipeline:

In the .gitlab-ci.yml file, define a job that needs access to the container registry.
Use environment variables for your GitLab username and the PAT.

Write the Job Script:

Use docker login command within the shell script of the CI/CD job.
Pass the credentials using -u <username> -p <token> options.

Here is a sample .gitlab-ci.yml configuration:

YAML
image: docker:latest

services:
  - docker:dind

stages:
  - build

build_image:
  stage: build
  script:
    - apk add --no-cache docker git
    - docker login -u $GITLAB_USER_NAME -p $CI_JOB_TOKEN registry.gitlab.com
    - # Further commands to push/pull images here

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, logging into the GitLab Container Registry involves creating a personal access token and configuring it within your CI/CD pipeline. Ensure that you use environment variables for credentials to keep secrets secure.

#### Common mistakes
include:

Not setting up the PAT with sufficient permissions.
Forgetting to add services: - docker:dind in .gitlab-ci.yml.
Using the wrong environment variable names or values.

For follow-ups, additional questions might involve scaling this setup across multiple jobs or managing credentials more securely, such as using GitLab’s secret variables or Kubernetes secrets management.

