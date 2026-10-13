---
type: "practice"
difficulty: "Intermediate"
---

# You need a GitHub Actions check that fails a PR if it enlarges a service's Docker image or dependency footprint past a threshold. Sketch how it compares against the base branch and where the base-state comparison can go wrong.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Isolating CI Environments](<../../../../TOOLS/Git_and_GitHub/Git_Core/Isolating_CI_Environments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

We'll create a GitHub Actions workflow that checks whether Pull Requests (PRs) increase the Docker image size or dependency footprint of a service beyond a predefined threshold. This involves comparing the state of the target branch with its base state and ensuring that PRs do not exceed certain limits to maintain codebase integrity.

#### Triaging the Question

The core issue here is ensuring that PRs do not introduce changes that increase the Docker image size or dependency footprint above acceptable thresholds. The workflow needs to compare the current state (target branch) against a baseline state (base branch). This comparison helps catch potential issues early in the development process, reducing the risk of bloated images and unnecessary dependencies.

#### Assumptions
We are working with a monorepo containing multiple services.
Docker image size is measured in bytes or megabytes.
Dependency footprint is tracked using package managers like npm, pip, etc.
The threshold values for image size and dependency count are configurable.
Step-by-Step Solution:
Define Thresholds: Establish the maximum allowed image size and the limit on the number of dependencies.
Set Up GitHub Actions Workflow:
Use a GitHub Actions workflow file, typically located in .github/workflows/.
Configure the workflow to run on specific events, such as push or pull_request.

#### Example
Workflow File (.github/workflows/image-size-and-dependency-check.yml):
YAML
name: Check Docker Image Size and Dependency Footprint

on:
  pull_request:

jobs:
  check-image-size-and-dependency-foo:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Get base branch state
        id: get_base_state
        run: |
          # Use git commands to clone the base state if necessary

      - name: Build and Tag Docker Image
        run: docker build -t my-service .

      - name: Calculate Current Dependency Footprint
        id: calc_current_deps
        run: npm ls | wc -l

      - name: Calculate Base Dependency Footprint
        id: calc_base_deps
        run: |
          # Use a script or tool to compare the current dependency footprint with the base state

      - name: Compare Image Sizes
        run: |
          local_size=$(docker images --format "{{.Size}}" my-service)
          base_size=$(get_base_image_size_from_log)  # Placeholder for actual size retrieval logic
          if ((local_size > base_size * THRESHOLD_MULTIPLIER)); then
            echo "Error: Docker image size exceeded threshold"
            exit 1
          fi

      - name: Compare Dependency Footprints
        run: |
          current_deps_count=$(calc_current_deps)
          base_deps_count=$(calc_base_deps)
          if ((current_deps_count > base_deps_count * DEP_THRESHOLD_MULTIPLIER)); then
            echo "Error: Dependency footprint exceeded threshold"
            exit 1
          fi

      - name: Success
        if: always()
        run: |
          echo "No errors detected in PR"
Explanation:
Get Base Branch State: This step ensures that the workflow has access to the original state of the base branch, which can be stored in a separate repository or fetched using Git commands.
Build and Tag Docker Image: The Docker image is built from the target branch code.
Calculate Current Dependency Footprint: This step uses npm ls (or an equivalent command for other package managers) to determine the current dependency count.
Compare with Base State: By comparing the current state against the base state, we can identify any changes that might have introduced new dependencies or increased image size.

#### Closing Summary

This GitHub Actions workflow effectively monitors PRs to ensure they do not introduce excessive Docker image sizes or dependencies. The key is accurately tracking and comparing states to maintain codebase integrity and performance.

#### Common Mistakes/Follow-Ups
Incorrect Baseline State: Ensuring the base state is correctly fetched and compared.
Inaccurate Threshold Values: Setting realistic thresholds that align with business requirements.

#### Performance Considerations
Optimizing the comparison process for large repositories to avoid long-running workflows.
Error Handling: Adding more robust error handling to manage potential issues during image building or dependency calculation.

**Code:**

```yaml
name: Check Docker Image Size and Dependency Footprint

on:
  pull_request:

jobs:
  check-image-size-and-dependency-foo:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Get base branch state
        id: get_base_state
        run: |
          git clone <base_branch_repo_url>

      - name: Build and Tag Docker Image
        run: docker build -t my-service .

      - name: Calculate Current Dependency Footprint
        id: calc_current_deps
        run: npm ls | wc -l

      - name: Calculate Base Dependency Footprint
        id: calc_base_deps
        run: |
          # Use a script or tool to compare the current dependency footprint with the base state

      - name: Compare Image Sizes
        run: |
          local_size=$(docker images --format "{{.Size}}" my-service)
          base_size=$(get_base_image_size_from_log)  # Placeholder for actual size retrieval logic
          if ((local_size > base_size * THRESHOLD_MULTIPLIER)); then
            echo "Error: Docker image size exceeded threshold"
            exit 1
          fi

      - name: Compare Dependency Footprints
        run: |
          current_deps_count=$(calc_current_deps)
          base_deps_count=$(calc_base_deps)
          if ((current_deps_count > base_deps_count * DEP_THRESHOLD_MULTIPLIER)); then
            echo "Error: Dependency footprint exceeded threshold"
            exit 1
          fi

      - name: Success
        if: always()
        run: |
          echo "No errors detected in PR"
```

**Data:**

```yaml
# Example of a base image size log (placeholder)
base_image_size_log:
  "service_name":
    local_size: 230MB
    base_size: 195MB
```

