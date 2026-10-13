---
type: "practice"
difficulty: "Deep"
---

# Your GitHub Actions matrix deploys to 6 regions in parallel and one region's deploy failed silently because the job didn't fail the workflow. How do you make partial matrix failures block promotion, and how do you avoid a green pipeline hiding a broken region?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Concurrency Groups and Merge Queues](<../../../../TOOLS/Git_and_GitHub/Git_Core/Concurrency_Groups_and_Merge_Queues.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of silent failures in GitHub Actions when running a matrix deployment across multiple regions, we need to ensure that any partial failure blocks the promotion process and prevents the overall pipeline from marking as green if even one region fails. Here’s how you can achieve this:

Understand the Current Issue: The current setup deploys to 6 regions in parallel using GitHub Actions’ matrix strategy. If a deployment to a single region fails, it doesn’t fail the workflow, which can lead to an incorrect "green" status.

Define the Goal: We need to ensure that if any deployment in the matrix fails, the entire pipeline is marked as failed and no further deployments are attempted until the issue is resolved.

#### First-Principles Solution

Use concurrency to control concurrent runs.
Implement a custom step to check each region’s status.
Use GitHub Actions’ failure() context to mark the workflow as a failure if any job fails.

Let's break down the solution step by step:

#### Triage

The question is probing your understanding of GitHub Actions, concurrency management, and how to handle partial failures in matrix jobs. The assumptions are that you are familiar with GitHub Actions syntax, matrix strategies, and the failure() context.

Step-by-Step Solution

Define Concurrency:

Use the concurrency directive to limit concurrent executions to a single region at a time.
YAML
concurrency: ${{ github.workflow }}-${{ matrix.region }}

jobs:
  deploy-region:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        region: [region1, region2, region3, region4, region5, region6]
      fail-fast: false  # This allows all regions to be attempted

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Deploy to ${matrix.region}
        env:
          REGION: ${{ matrix.region }}
        run: |
          echo "Deploying to region: $REGION"
          # Your deployment command here
          if [ $? != 0 ]; then
            echo "::set-output name=failed::true"
            exit 1
          fi

      - name: Check Deployment Status
        if: always() && needs.deploy-region.outputs.failed == 'true'
        run: |
          echo "Deployment to region ${REGION} failed. Stopping further deployments."
          # Optionally, you can add logging or notifications here
          exit 1

Use failure() Context:

In the matrix job steps, use conditional logic to set an output variable if a deployment fails.
Use the needs keyword to control the workflow based on previous step outcomes.

Conditional Logic and Failure Handling:

YAML
jobs:
  deploy-region:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        region: [region1, region2, region3, region4, region5, region6]
      fail-fast: false

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Deploy to ${matrix.region}
        env:
          REGION: ${{ matrix.region }}
        run: |
          echo "Deploying to region: $REGION"
          # Your deployment command here
          if [ $? != 0 ]; then
            echo "::set-output name=failed::true"
            exit 1
          fi

      - name: Check Deployment Status
        if: always() && needs.deploy-region.outputs.failed == 'true'
        run: |
          echo "Deployment to region ${REGION} failed. Stopping further deployments."
          # Optionally, you can add logging or notifications here
          exit 1

#### Closing Summary
and Common Mistakes/Follow-Ups

By implementing the above steps, we ensure that if any deployment in the matrix fails, it will immediately stop all other deployments and mark the workflow as a failure. This prevents the pipeline from showing a green status when there is an issue.

#### Common Mistakes

Forgetting to set the failed output variable.
Not using the if: always() condition properly to control workflow logic.
Misusing the needs keyword or not understanding its implications on workflow execution.

If you have follow-up questions, consider discussing alternative approaches like using multiple workflows or more advanced error handling techniques. This solution provides a robust mechanism for managing partial failures and ensures that all regions are tested thoroughly before promotion can occur.

**Code:**

```yaml
jobs:
  deploy-region:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        region: [region1, region2, region3, region4, region5, region6]
      fail-fast: false

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Deploy to ${matrix.region}
        env:
          REGION: ${{ matrix.region }}
        run: |
          echo "Deploying to region: $REGION"
          # Your deployment command here
          if [ $? != 0 ]; then
            echo "::set-output name=failed::true"
            exit 1
          fi

      - name: Check Deployment Status
        if: always() && needs.deploy-region.outputs.failed == 'true'
        run: |
          echo "Deployment to region ${REGION} failed. Stopping further deployments."
          # Optionally, you can add logging or notifications here
          exit 1
```

