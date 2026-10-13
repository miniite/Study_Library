---
type: "practice"
difficulty: "Deep"
---

# Two concurrent CI runs for the same Spark-job PR both spin up an expensive EMR cluster and can push conflicting artifacts. Explain concurrency groups and cancel-in-progress, and how to configure them so a real production backfill deploy is never cancelled mid-flight.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Concurrency Groups and Merge Queues](<../../../../TOOLS/Git_and_GitHub/Git_Core/Concurrency_Groups_and_Merge_Queues.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of concurrent CI runs for the same Spark job PR spinning up expensive EMR clusters and potentially pushing conflicting artifacts, we need to understand concurrency groups and cancel-in-progress features in GitHub Actions. These features help manage parallel execution of workflows while ensuring that certain actions do not interfere with each other.

#### Triaging the Question

The core challenge here is managing concurrent CI runs so that they don’t conflict with each other, specifically focusing on preventing multiple EMR clusters from being spun up simultaneously and ensuring that ongoing backfill deployments are not interrupted. The key aspects we need to cover include:

Understanding concurrency groups.
Configuring cancel-in-progress.
Ensuring seamless production deployments.

#### First-Principles Solution
Concurrency Groups

Concurrent groups in GitHub Actions allow us to define workflows such that only one job from a specified group can run at any time. This is achieved by setting the concurrency property in the workflow file.

Here’s an example configuration:

YAML
on:
  push:
    branches: [ main ]
jobs:
  spark-job:
    runs-on: self-hosted
    concurrency:
      group: spark-jobs
      cancel-in-progress: true
    steps:
      # Spark job steps here

This configuration ensures that only one spark-job can run at a time, even if multiple pull requests are pushed to the repository.

Cancel-In-Progress

Cancel-in-progress is a feature that allows workflows to be cancelled when another workflow from the same concurrency group starts. This helps prevent overlapping or conflicting operations.

To enable cancel-in-progress:

YAML
concurrency:
  group: spark-jobs
  cancel-in-progress: true

By setting cancel-in-progress to true, we ensure that if a new CI run for the same Spark job is initiated, it will automatically cancel any in-progress runs from the same concurrency group.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, by configuring concurrency groups with cancel-in-progress: true, we can effectively manage concurrent CI runs for Spark jobs. This ensures that only one EMR cluster is spun up at a time and prevents conflicting artifacts from being pushed to production.

#### Common mistakes
include:

Not setting the correct concurrency group name.
Failing to enable cancel-in-progress, leading to overlapping operations.

#### Follow-ups
might include questions about how this configuration interacts with other GitHub Actions workflows or how it handles edge cases like race conditions during deployment.

**Code:**

```yaml
on:
  push:
    branches: [ main ]
jobs:
  spark-job:
    runs-on: self-hosted
    concurrency:
      group: spark-jobs
      cancel-in-progress: true
    steps:
      - name: Checkout code
        run: git checkout .
      - name: Build and deploy Spark job
        # Spark job deployment steps here
```

**Flow:**

1. Trigger PR
2. CI Run 1
3. CI Run 2
4. (Start Spark Job)
5. (Start Spark Job)
6. (Spin up EMR Cluster 1)
7. (Check Concurrency Group)
8. (Cancel In-Progress) OR(Proceed to Spin up EMRCluster 2)

