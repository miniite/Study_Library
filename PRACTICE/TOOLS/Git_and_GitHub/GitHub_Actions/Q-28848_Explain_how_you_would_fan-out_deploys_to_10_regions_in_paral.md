---
type: "practice"
difficulty: "Deep"
---

# Explain how you would fan-out deploys to 10 regions in parallel and fail the overall run if any single region deploy fails.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Concurrency Control in Workflows](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Concurrency_Control_in_Workflows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To fan-out deploys to 10 regions in parallel using GitHub Actions while ensuring that the overall workflow fails if any single region deployment fails, we can use multiple jobs with a conditional structure that triggers on success and aggregates the results. This setup ensures that the entire pipeline only succeeds when all regional deployments are successful.

#### Triaging the Question

The question is about orchestrating concurrent deployments across 10 regions using GitHub Actions and ensuring the overall workflow's failure if any region fails its deployment. The key points to address are:

Parallel execution of deployments in multiple regions.
Conditional logic to fail the entire pipeline based on individual job success or failure.

#### First Principles Solution

To achieve this, we will use a single main job that triggers 10 sub-jobs (one for each region). Each sub-job will handle the deployment in its respective region. If any of these sub-jobs fails, the main job should fail, and thus the overall pipeline.

Here's how you can structure it:

Step-by-Step Solution
Define a Matrix Strategy: Use GitHub Actions' matrix strategy to parallelize the deployments across regions.
Sub-Job for Each Region: Create separate jobs for each region with conditional logic to fail the main job if any sub-job fails.
Detailed Implementation
YAML
name: Multi-Region Deployment

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  deploy-regions:
    name: Deploy in <region>
    runs-on: ubuntu-latest
    strategy:
      matrix:
        regions: ["region1", "region2", "region3", "region4", "region5",
                  "region6", "region7", "region8", "region9", "region10"]
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Node.js (Example for Node.js)
        run: |
          echo "Node.js setup"

      - name: Deploy to region ${matrix.regions}
        run: |
          # Example command to deploy to a specific region
          echo "Deploying to ${{ matrix.regions }}"
          if ! npm run deploy --region=${{ matrix.regions }}; then
            exit 1
          fi

#### Closing Summary
and Common Mistakes
Summary: The solution uses GitHub Actions' matrix strategy to parallelize deployments across regions. Each region’s deployment is handled by a separate job, with the main job failing if any of these jobs fail.

#### Common Mistakes
Failing to define a matrix strategy properly: Not using strategy and matrix.
Ignoring conditional logic in sub-jobs: Missing the check for success/failure at the end of each sub-job.
Additional Considerations
Parallelism: Ensure that your deployment scripts are idempotent and can be safely run multiple times.
Retry Mechanisms: Implement retry logic within individual jobs to handle transient issues without failing the entire pipeline.
Monitoring: Use GitHub Actions' built-in notifications or external tools for monitoring the status of deployments.

**Code:**

```yaml
name: Multi-Region Deployment

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  deploy-regions:
    name: Deploy in <region>
    runs-on: ubuntu-latest
    strategy:
      matrix:
        regions: ["region1", "region2", "region3", "region4", "region5",
                  "region6", "region7", "region8", "region9", "region10"]
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Node.js (Example for Node.js)
        run: |
          echo "Node.js setup"

      - name: Deploy to region ${matrix.regions}
        run: |
          # Example command to deploy to a specific region
          echo "Deploying to ${{ matrix.regions }}"
          if ! npm run deploy --region=${{ matrix.regions }}; then
            exit 1
          fi
```

