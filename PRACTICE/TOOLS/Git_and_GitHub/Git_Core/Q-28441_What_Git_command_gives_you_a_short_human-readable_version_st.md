---
type: "practice"
difficulty: "Intermediate"
---

# What Git command gives you a short, human-readable version string based on the nearest tag, and why is it useful in builds?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Commit Messages](<../../../../TOOLS/Git_and_GitHub/Git_Core/Commit_Messages.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To answer this question, we need to understand the git describe command. This command provides a short, human-readable version of your commit, which includes the closest tag and the number of commits since that tag. It's incredibly useful in builds because it helps ensure reproducibility by linking a specific build with a known software version.

#### Triage
What is git describe?

The git describe command is used to determine the current branch’s latest tag, along with the number of commits after that tag and the short SHA hash. It's particularly useful in continuous integration (CI) systems where you want to know exactly which version of a software package was built.

First Principles: Why Use git describe?

In many development environments, especially in CI/CD pipelines, it's crucial to have clear visibility into what exact code is being used for a build. This helps in maintaining reproducibility and debugging issues later on. Here’s how the command works:

Identify Closest Tag: It starts by identifying the closest tag available.
Count Commits After Tag: It counts how many commits have occurred after that tag.
SHA Hash: It appends a short SHA hash of the commit, ensuring fine-grained resolution.

#### Example
Use Case in Builds

Imagine you are setting up a CI pipeline for a web application using GitHub Actions. You might use git describe to generate a unique version string based on your last tagged release and the current commit. Here’s how it could look:

YAML
# GitHub Action Workflow
name: Build and Deploy

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v2

      - name: Set up Node.js environment
        uses: actions/setup-node@v2
        with:
          node-version: '14.x'

      - name: Install dependencies
        run: npm install

      - name: Build the application
        run: npm run build

      - name: Get Git Tag Info
        id: describe
        run: git describe --tags --always --dirty

      - name: Store Version in Environment Variable
        env:
          VERSION: ${{ steps.describe.outputs.result }}
        run: echo "VERSION=$VERSION" >> .env

# In your application, use the version from the environment variable

#### cons
ole.log(`Using version ${process.env.VERSION}`);
Output of git describe

For example, if you have a tag v1.0 and there are 5 commits since that tag, git describe --tags might output something like:

CODE
v1.0-5-gabc1234

Here:

v1.0: The closest tag.
-5-: The number of commits after the tag.
gabc1234: The short SHA hash.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Using git describe ensures that your build scripts can accurately reflect the exact state of the codebase. This is particularly useful in environments where multiple versions might be deployed or rolled back, ensuring you always know which version was built.

#### Common mistakes
include not handling tags correctly (e.g., if no tag exists) and ignoring the --dirty flag to indicate uncommitted changes.

#### Follow-up
questions could include:

How does git describe handle branches without any tagged commits?
What are some alternative commands or strategies for versioning in Git?

**Code:**

```yaml
# GitHub Actions Workflow Example
name: Build and Deploy

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v2

      - name: Set up Node.js environment
        uses: actions/setup-node@v2
        with:
          node-version: '14.x'

      - name: Install dependencies
        run: npm install

      - name: Build the application
        run: npm run build

      - name: Get Git Tag Info
        id: describe
        run: git describe --tags --always --dirty

      - name: Store Version in Environment Variable
        env:
          VERSION: ${{ steps.describe.outputs.result }}
        run: echo "VERSION=$VERSION" >> .env
```

**Example input / output:**

```text
# Example Output of `git describe --tags --always --dirty`
v1.0-5-gabc1234-dirty
```

