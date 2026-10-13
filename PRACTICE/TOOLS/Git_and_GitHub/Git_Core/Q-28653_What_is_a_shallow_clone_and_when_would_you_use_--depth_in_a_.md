---
type: "practice"
difficulty: "Deep"
---

# What is a shallow clone and when would you use --depth in a CI pipeline?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Shallow Clones in CI](<../../../../TOOLS/Git_and_GitHub/Git_Core/Shallow_Clones_in_CI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To start, let’s quickly define what a shallow clone is and why it might be useful in a Continuous Integration (CI) pipeline.

TL;DR:

A shallow clone limits the history of a Git repository, downloading only recent commits to save time and bandwidth. This can be particularly beneficial in CI pipelines where full clones are unnecessary and slow down builds.

Detailed Explanation:
What is a Shallow Clone?

A shallow clone is a version of a Git repository that doesn't contain the entire commit history from the origin repository. Instead, it only includes commits up to a certain depth (defined by --depth), along with their metadata like tags and branch information.

In a CI pipeline, using a shallow clone can significantly reduce initial download time and bandwidth usage. This is especially useful when the full history of the repository is not required for building and testing.

Why Use --depth in a CI Pipeline?
Bandwidth Optimization: Full clones can be very large, which means they consume significant bandwidth during the initial checkout phase. In environments with limited internet connectivity or high costs, this can be a major bottleneck.
Time Efficiency: A shallow clone is faster to download and initialize compared to a full clone. This can speed up the CI pipeline start-up time.
Cost Reduction: In cloud-based CI/CD platforms, where bandwidth usage often comes with additional costs, using a shallow clone helps in reducing these expenses.
How to Use --depth in GitHub Actions

Here’s an example of how you might configure a GitHub Action to use a shallow clone:

YAML
name: Build and Test

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout with Shallow Clone
      uses: actions/checkout@v3
      with:
        fetch-depth: 2  # Fetch only the last two commits from the default branch

In this example, fetch-depth is set to 2, meaning it will download only the latest two commits. You can adjust the value based on your requirements.

Edge Cases and Considerations
Dependency Management: If your application has dependencies that are not in the shallow clone’s commit history, you might need a full clone for those dependencies.
Long-Term Use: While shallow clones save initial download time, they may require periodic updates to ensure that all necessary changes are included over time.

#### Closing Summary
and Common Mistakes

Using --depth effectively can greatly enhance the performance of your CI pipelines by reducing bandwidth usage and startup times. However, it's important to consider whether the trade-off in history completeness meets your needs. A common mistake is not considering this option at all when dealing with large repositories.

Final Check

This answer covers the core concept of shallow clones, their utility in CI pipelines, and provides a practical example using GitHub Actions. All relevant details have been included, and no additional elements are needed.

