---
type: "practice"
difficulty: "Intermediate"
---

# What is the GitLab Container Registry and how do you push an image to it from a pipeline?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Docker Layer Caching](<../../../../TOOLS/GitLab/GitLab_CI/Docker_Layer_Caching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: The GitLab Container Registry (GCR) is a service provided by GitLab that allows users to store, manage, and deploy container images. To push an image to GCR from a pipeline, you need to configure the Docker credentials in your GitLab project settings, write a CI/CD configuration file, and use the docker command within your pipeline script.

#### Triage
The question is asking about the GitLab Container Registry (GCR) and how it can be integrated with pipelines for pushing container images. The core aspects to cover are understanding GCR, setting up credentials, and writing a CI/CD configuration file that includes steps to push the image.

Step-by-Step Solution:

Understand GitLab Container Registry: GCR is a managed service by GitLab that lets you store your container images securely. It integrates seamlessly with GitLab's CI/CD pipeline system.
Set Up Docker Credentials: To interact with GCR, you need to configure the appropriate credentials in your GitLab project settings or within the pipeline itself.
Write CI/CD Configuration File:
Define a .gitlab-ci.yml file in your repository root directory.
Configure job steps to build and push container images.

Here’s an example configuration:

YAML
# .gitlab-ci.yml

stages:
  - build
  - deploy

build_image:
  stage: build
  script:
    - docker build -t myapp:$CI_COMMIT_REF_SLUG .
    - echo "$MY_GCR_PASSWORD" | docker login --username="$MY_GCR_USERNAME" --password-stdin https://gitlab.example.com/registry
    - docker tag myapp:$CI_COMMIT_REF_SLUG https://gitlab.example.com/namespace/myapp:latest
    - docker push https://gitlab.example.com/namespace/myapp:latest

deploy:
  stage: deploy
  script:
    # Deployment steps
Push the Image:
The docker build command builds the Docker image.
echo "$MY_GCR_PASSWORD" | docker login --username="$MY_GCR_USERNAME" --password-stdin https://gitlab.example.com/registry logs in to GCR using environment variables.
docker tag tags the built image with a name that matches your registry path.
docker push pushes the tagged image to GCR.

#### Closing Summary
Pushing container images to the GitLab Container Registry involves setting up credentials, writing CI/CD configuration files, and running appropriate commands within the pipeline. Proper setup ensures secure and automated deployment of containerized applications.

#### Common Mistakes/Follow-Ups

Forgetting to set the Docker credentials.
Misconfiguring the image tagging or pushing process.
Not updating the .gitlab-ci.yml file to include all necessary steps.
Security concerns related to handling sensitive information like passwords directly in code. Consider using GitLab's secrets feature for better security.

**Code:**

```yaml
stages:
  - build
  - deploy

build_image:
  stage: build
  script:
    - docker build -t myapp:$CI_COMMIT_REF_SLUG .
    - echo "$MY_GCR_PASSWORD" | docker login --username="$MY_GCR_USERNAME" --password-stdin https://gitlab.example.com/registry
    - docker tag myapp:$CI_COMMIT_REF_SLUG https://gitlab.example.com/namespace/myapp:latest
    - docker push https://gitlab.example.com/namespace/myapp:latest

deploy:
  stage: deploy
  script:
    # Deployment steps
```

