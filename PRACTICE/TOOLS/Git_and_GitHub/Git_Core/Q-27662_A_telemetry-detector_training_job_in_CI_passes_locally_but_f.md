---
type: "practice"
difficulty: "Intermediate"
---

# A telemetry-detector training job in CI passes locally but fails on the runner with 'dubious ownership' and missing-tag errors. Explain how CI's root-owned, shallow, detached-HEAD checkout causes each error and the specific fix for each in an ML-on-telemetry pipeline.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Shallow Clones in CI](<../../../../TOOLS/Git_and_GitHub/Git_Core/Shallow_Clones_in_CI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

In this scenario, the telemetry-detector training job is failing on a CI runner due to two specific issues: 'dubious ownership' and missing tags. These errors arise from how Git checks out code in shallow mode with a detached HEAD, which is common in CI environments where multiple jobs may need access to different parts of a repository without modifying it fully. The fixes involve ensuring proper permissions for the runner and fetching all required tags.

#### Triaging the Question

The question revolves around understanding why a training job that passes locally fails on a CI runner due to 'dubious ownership' and missing-tag errors. This issue is rooted in how Git handles shallow clones with detached HEAD states, which are typical in CI/CD pipelines where multiple jobs may require read-only access to specific parts of the repository.

#### First-Principles Solution
Step 1: Understanding Shallow Clones and Detached HEAD

A shallow clone is a lighter version of the full Git repository, containing only a portion of history. In CI environments, shallow clones are often used to reduce storage and download times, but this can lead to issues if multiple jobs need different branches or tags.

When a job uses git checkout -b <branch-name> --detach <commit-hash> (or similar commands), it ends up in a detached HEAD state where the working directory is pointing at a specific commit without any branch pointers. This setup doesn't allow for operations like tag checking, which can lead to issues if tags are required by the job.

Step 2: Analyzing 'Dubious Ownership' Errors

The error "dubious ownership" typically indicates that Git could not properly determine the owner of a file or directory due to shallow clone limitations. In a shallow clone, some files and directories might not have their proper permissions or attributes set correctly because they are incomplete.

To fix this issue:

Ensure the CI runner has appropriate read permissions for all necessary files in the shallow clone.

#### Cons
ider using a full clone instead of a shallow one if ownership is critical to your application logic. Alternatively, you can use git clone --mirror which allows copying the entire repository but may not be suitable due to its size and complexity.
Step 3: Handling Missing Tag Errors

Missing-tag errors occur when a job requires specific tags that are not available in the shallow clone or detached HEAD state. These tags might be essential for certain parts of your ML pipeline, such as versioning models or dependencies.

To fix this:

Use git fetch --tags before running operations that require tags. This command ensures all tags from the remote repository are fetched.
Alternatively, if you cannot use full clones due to size constraints, ensure that the CI runner has a mechanism to dynamically fetch necessary tags just before they are needed in the job.

#### Closing Summary
and Common Mistakes

In summary, the key issues here are related to Git's shallow clone limitations and how detached HEAD states affect operations like tag checking. The solutions involve either modifying repository permissions or adjusting Git commands to ensure all required data is available during the CI job execution.

#### Common mistakes
include assuming that a shallow clone will always be sufficient for complex pipelines requiring specific branches, tags, or ownership attributes. Always consider the trade-offs between reducing download times and ensuring complete functionality in your CI setup.

Common Follow-Ups
How do you balance download time with the completeness of the Git checkout?
Are there scenarios where using a full clone might be more appropriate despite increased storage costs?
What security considerations should be taken into account when granting permissions to CI runners?

**Code:**

```bash
# Example command to ensure tags are fetched before running the job
git fetch --tags
```

