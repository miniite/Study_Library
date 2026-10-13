---
type: "practice"
difficulty: "Intermediate"
---

# How do you build a Docker image inside a GitLab CI pipeline using docker-in-docker (dind)?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Docker Layer Caching](<../../../../TOOLS/GitLab/GitLab_CI/Docker_Layer_Caching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build a Docker image inside a GitLab CI pipeline, we will use the docker-in-docker (DIND) method. This approach allows us to run Docker commands within a GitLab CI job without needing to configure a separate host for Docker. Here’s how you can set it up from first principles:

Understanding DIND: The idea behind DIND is that we start a container which runs the Docker daemon, and this container can then be used by other jobs in the pipeline to build and run Docker images.

Setting Up GitLab CI Configuration: We need to configure our .gitlab-ci.yml file to use the docker:dind service provided by GitLab runners. This service will start a Docker daemon inside a container, which can be used to build other images.

Step-by-Step Solution:

Define a job in your .gitlab-ci.yml that uses the docker image and specifies the dind service.
Write a script or commands to build your Docker image within this job.

Here’s an example of how you might write such a configuration:

YAML
stages:
  - build

.build-docker-image:
  stage: build
  services:
    - docker:dind
  script:
    - docker image ls # Ensure the Docker daemon is up and running.
    - docker pull <base-image> # Optionally, pull any base images you need.
    - docker build -t my-app-image . # Build your application Docker image.

#### Closing Summary
By using DIND in GitLab CI, we can effectively create and manage Docker images within our pipeline without the overhead of setting up a separate host for Docker.

#### Common Mistakes/Follow-Ups

Not Using services Keyword: Failing to specify the services keyword in your job configuration can prevent DIND from starting.
Missing Docker Daemon Initialization: Ensuring that the Docker daemon is initialized properly, as shown by docker image ls.
Permissions and Network Issues: Pay attention to network configurations if you’re running multiple services or accessing external resources.

**Flow:**

1. Define GitLab CI Pipeline
2. Specify DIND Service
3. Write Build Script for DockerImage
4. Test Docker Daemon
5. BUILD COMPLETE

