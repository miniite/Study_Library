---
type: "practice"
difficulty: "Deep"
---

# A CI job using a shallow clone breaks git describe and blame-based coverage tooling. Explain why depth-limited clones lose that info and how you'd fix it without giving up all the speed benefit.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Shallow Clones in CI](<../../../../TOOLS/Git_and_GitHub/Git_Core/Shallow_Clones_in_CI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're dealing with a CI job using shallow clones to optimize Git repository cloning times. However, these shallow clones lack certain information that deep clones retain, such as full commit history and blame data used by tools like git describe and coverage reporting tools. To understand why this happens and how to fix it without sacrificing the speed benefits of shallow clones, we need to delve into the core principles of Git cloning and repository structure.

Shallow clones are a feature in Git that allows for quick initial clone operations by downloading only a portion of the commit history. While they save time during setup, they also limit the amount of information available locally. Specifically, git describe and blame-based tools rely on the complete commit history, which is not present in shallow clones.

To address this issue without completely reverting to full clones, we can leverage Git’s capabilities for fetching additional history selectively or store necessary data separately. Here's a step-by-step approach:

Understand the Problem:

Shallow clones have limited depth and do not contain all commit history.
git describe and blame-based tools require full history to function correctly.

Identify Solutions:

Fetch additional commits as needed during CI jobs.
Store necessary metadata separately for quick access.

Fetch Additional Commits:

Use Git's --depth option in a strategic manner to fetch specific branches or tags.
Implement a script that runs at the start of the CI job to fetch missing data.

Store Metadata Separately:

Store blame and describe information in an external database or file.
Periodically update this metadata when pushing changes, ensuring it is available for CI jobs.

Implementing the Solution:

Use Git hooks to automate fetching of necessary commits during CI runs.
Create a CI step that updates blame and describe data if needed.
Store this information in a lightweight format like JSON or YAML.

Here’s an example implementation outline:

BASH
# Example Bash script for fetching additional commits
#!/bin/bash

# Define the repository URL and shallow clone depth
REPO_URL="https://github.com/your/repo.git"
SHALLOW_DEPTH=50 # Adjust as necessary

# Clone the repository with a shallow depth
git clone --depth $SHALLOW_DEPTH $REPO_URL

# Fetch additional branches or tags
git fetch origin --tags
git fetch origin branch1:branch1

By running this script at the beginning of your CI job, you can ensure that all necessary commits are available for git describe and blame-based tools. This approach maintains the speed benefits of shallow clones while providing full functionality.

