---
type: "practice"
difficulty: "Deep"
---

# A dependency-bump PR is green in CI but breaks a downstream service that consumes your library only at runtime; how do you catch cross-repo breakage before it ships?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Schema Changes in Delivery Pipelines](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Schema_Changes_in_Delivery_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of catching cross-repo breakage before it ships, we need to ensure our Continuous Integration/Continuous Deployment (CI/CD) pipeline is robust and comprehensive enough to detect runtime issues that might arise from dependency bumps. The crux of the problem lies in understanding how CI can simulate a real-world scenario where dependencies are updated at runtime, which isn't directly covered by traditional unit tests or static analysis.

#### Triage

The question is probing on two primary fronts:

Understanding Dependency Management: How we handle and test changes to our library's dependencies.
Runtime Validation: Ensuring that the changes don’t break downstream services only at runtime, which traditional CI steps might not fully capture.

We'll assume a common Git flow where dependency bumps are made in one repository (let’s call it RepoA), and another repository (RepoB) depends on RepoA. The issue here is that while the PR from RepoA passes its own CI checks, there's a runtime failure when RepoB attempts to use the updated library.

#### First-Principles Solution
Simulate Real-World Usage: Create a mechanism where RepoA’s CI can simulate how RepoB uses it at runtime.
Runtime Environment Setup: Use containerization (e.g., Docker) to set up an environment that mirrors the actual deployment scenario, including dependencies and configurations.
Step-by-Step Solution
Containerize RepoA: Build a Docker image for RepoA. This image should include all its runtime dependencies and any necessary configuration files.
Mount RepoB Dependencies: In the CI pipeline of RepoA, mount or copy the source code of RepoB into the container, so it can be executed as part of the test suite.
Run Integration Tests in Container: Write integration tests that run within the Docker container to simulate how RepoB uses RepoA. These tests should cover various use cases and edge scenarios.
Automate Deployment Simulation: Use deployment tools like Helm or Kubernetes inside the container to simulate a production-like deployment, ensuring all runtime interactions are accurately tested.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By implementing this solution, we ensure that any cross-repo breakage is caught before it reaches production. The key takeaway is the importance of runtime validation and how traditional unit tests might not cover all scenarios.

#### Common Mistakes

Overlooking the need for a comprehensive runtime environment setup.
Failing to include critical dependencies or configurations in the containerized test scenario.
Not adequately simulating real-world use cases, leading to false positives or negatives.

#### Follow-Ups

Discussing how often these tests should be run (e.g., nightly builds) and their integration with other CI pipelines.
Exploring alternative testing strategies like canary releases if full runtime validation is resource-intensive.

**Comparison:**

```
Test Case	Expected Result	Actual Result
Logging	No errors	None
Metrics	Correct values	Incorrect
```

**Code:**

```python
# Example Python script for a Dockerfile in RepoA
FROM python:3.9

COPY . /app
WORKDIR /app

RUN pip install -e .
```

**Example input / output:**

```text
**Input Example:**
- `RepoA` PR with updated library dependency.
- Docker container image built from the CI pipeline.

**Output Example:**
- Integration tests in the Docker container pass, indicating no runtime breakage.
- If a test fails, pinpoint and fix the issue before merging to main.
```

**Data:**

```yaml
repos:
  repo_a:
    dependencies:
      - repo_b
  repo_b:
    dependencies:
      - repo_a
```

**Flow:**

1. Build RepoA Docker Image
2. Mount RepoB Source inContainer
3. Integration Tests Run WithinContainer
4. Runtime Validation Pass/Fail
5. Fix and Re-test or Merge toMain

