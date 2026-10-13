---
type: "practice"
difficulty: "Intermediate"
---

# A training job in CI uses a shallow clone for speed, and git describe fails so your model version string comes out wrong. Explain why depth-limited clones lose tag reachability and how you'd fix versioning without cloning full history.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Shallow Clones in CI](<../../../../TOOLS/Git_and_GitHub/Git_Core/Shallow_Clones_in_CI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Git repositories in a continuous integration (CI) environment, shallow clones can be used to speed up operations by only fetching a subset of the repository's history. However, this comes at the cost of losing access to certain branches and tags that are not included in the shallow clone. In your scenario, using a shallow clone leads to issues with git describe, which relies on all available tags in the history for generating meaningful model version strings.

To understand why shallow clones lose tag reachability, it's important to grasp how Git handles histories. A shallow clone only fetches commits up to a certain depth from the tip of the main branch (often master or main). Tags are not automatically fetched and must be explicitly pulled. If critical tags are not included in the initial shallow clone, they will not be accessible through normal operations like git describe.

To fix this issue without cloning full history, we need to ensure that all necessary tags are available within the CI environment. Here’s a step-by-step approach:

Explicitly Fetch Tags: Modify your Git workflow to include fetching of tags as part of the initial shallow clone process or as a separate step after the shallow clone is created.
Tag Management in Repository Settings: Ensure that relevant branches and tags are tagged properly and pushed to the remote repository so they can be fetched by CI jobs.

Let's break down these steps:

Step 1: Explicitly Fetch Tags

When creating a shallow clone, you can use --depth with --recurse-submodules=on-demand or explicitly fetch required branches and tags. However, for tags specifically, using git fetch --tags will ensure that all available tags are fetched.

SH
# Create a shallow clone with depth 1 (for example)
git clone https://github.com/example/repo.git --depth 1

# Fetch all tags from the remote repository
cd repo
git fetch --tags

This command ensures that even if you started with a shallow clone, fetching --tags will add all necessary tag information to your local environment.

Step 2: Tag Management in Repository Settings

Ensure that tags are created and pushed correctly. In Git, creating a tag is done using the following commands:

SH
# Create a new annotated tag (optional)
git tag -a v1.0 -m "Version 1.0"

# Push the tag to the remote repository
git push origin v1.0

For automated versioning in CI, you might want to use a script or a Git hook to automate this process:

SH
#!/bin/bash

# Check out the correct branch
git checkout main

# Fetch all tags from the remote repository
git fetch --tags

# Use git describe to generate the version string
version=$(git describe --tags --abbrev=0)

echo "Model Version: $version"

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, shallow clones can cause issues with git describe because they lack full history. To mitigate this, ensure that tags are explicitly fetched or included in the clone process. Also, manage tags properly within your repository settings.

#### Common mistakes
include not recognizing the need to fetch tags separately and assuming all necessary information is available by default in a shallow clone. Follow-ups could involve discussing how different CI tools handle such situations and potential trade-offs between speed and completeness of Git history.

