---
type: "practice"
difficulty: "Deep"
---

# Docker-in-docker builds are slow and eat runner disk. What alternatives would you evaluate and what are their trade-offs?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Docker Layer Caching](<../../../../TOOLS/GitLab/GitLab_CI/Docker_Layer_Caching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of slow Docker-in-Docker (DIND) builds and excessive runner disk usage in GitLab CI, we need to explore alternative approaches that can mitigate these problems while considering various trade-offs such as performance, resource utilization, and complexity.

#### Triage

The question is asking for alternatives to DIND builds within GitLab CI/CD pipelines. The key areas of consideration are:

Performance: Faster build times.
Resource Utilization: Efficient use of runner disk space.
Complexity: How easy or difficult the alternative solutions are to set up and manage.

#### First-Principles Solution
1. Pre-Built Images

Concept: Use pre-built Docker images for jobs that require a full container stack, thereby avoiding DIND.

Why Use It? Saves time as you don’t need to spin up a new Docker instance for each job.

#### Trade-offs
Performance: Faster builds due to no overhead of DIND setup.
Resource Utilization: Less disk usage since images are reusable across multiple jobs.
Complexity: Requires careful management and potentially larger initial image sizes.
2. Multi-Stage Builds

Concept: Use multi-stage Dockerfile builds in GitLab CI/CD, where stages share the same runner, but build different artifacts.

Why Use It? Reduces disk usage by reusing the base environment across multiple jobs.

#### Trade-offs
Performance: Potentially slower if the setup and teardown of each stage are complex.
Resource Utilization: Efficient use of storage since images from previous stages can be reused.
Complexity: Requires a different Dockerfile strategy but can simplify runner management.
3. Kubernetes Runner

Concept: Use a GitLab CI runner that runs directly on a Kubernetes cluster, leveraging container orchestration for resource isolation and management.

Why Use It? Provides better scalability, resource utilization, and flexibility in managing complex jobs.

#### Trade-offs
Performance: Faster builds as there is no DIND overhead; containers are run natively on the runner nodes.
Resource Utilization: Efficient use of resources since pods can be managed at a cluster level.
Complexity: Higher setup and management complexity, especially for non-expert users.
4. Docker Swarm

Concept: Use Docker Swarm as a runner orchestration layer to manage containerized jobs without DIND.

Why Use It? Provides more granular control over resources and can be integrated with Kubernetes.

#### Trade-offs
Performance: Faster builds due to reduced overhead of DIND.
Resource Utilization: Efficient resource management within Swarm.
Complexity: Requires setup and maintenance of a Docker Swarm environment.

#### Closing Summary

Each alternative has its own strengths and weaknesses, making them suitable for different scenarios. Pre-built images and multi-stage builds are simpler to implement but may not scale as well in complex environments. Kubernetes and Docker Swarm offer more advanced features but come with additional complexity in setup and management.

#### Common Mistakes/Follow-Ups
Mistake: Overlooking the performance benefits of pre-built images.

#### Follow-Up
Discussing how multi-stage builds can be implemented in a GitLab CI/CD pipeline example.
Mistake: Assuming that Kubernetes or Docker Swarm are always better without considering the additional complexity and setup effort.

**Comparison:**

```
Approach	Performance	Resource Utilization	Complexity
Pre-Built Images	Faster builds	Less disk usage	Lower
Multi-Stage Builds	Potentially slower setup	Efficient use of storage	Moderate
Kubernetes Runner	Faster builds	Efficient resource management	Higher
Docker Swarm	Faster builds	Efficient resource management	Higher
```

**Code:**

```yaml
# Example of using pre-built images in GitLab CI/CD
image: "my-prebuilt-image:latest"

stages:
  - build
  - test

build-job:
  stage: build
  script:
    - echo "Building the project..."
```

**Example input / output:**

```
Input: A GitLab CI/CD pipeline with DIND builds.
Output: Replaced with pre-built images, multi-stage Dockerfile, Kubernetes runner, or Docker Swarm setup.
PLAINTEXT
Copy
Using Pre-Built Images:
1. Define the image in the `image` section of the `.gitlab-ci.yml`.
2. Run jobs without setting up a new container for each job.
3. Faster builds and reduced disk usage.

Example Pipeline with Multi-Stage Builds:
- Stages: Build, Test, Deploy
- Use shared base images across stages to reduce disk usage and improve performance.
```

**Data:**

```yaml
# Example multi-stage Dockerfile
FROM base-image AS builder
WORKDIR /app
COPY . .
RUN make

FROM runtime-image
WORKDIR /app
COPY --from=builder /app/dist .

CMD ["start-app"]
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Pre-Built Images
6. Are jobs complex?
7. Maintain separate base images
8. Use a single pre-built image
9. Are you familiar withKubernetes?
10. Use Kubernetes runner
11. Consider Docker Swarm foradvanced features

