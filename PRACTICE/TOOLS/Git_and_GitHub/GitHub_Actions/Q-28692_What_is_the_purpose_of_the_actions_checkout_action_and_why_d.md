---
type: "practice"
difficulty: "Basic"
---

# What is the purpose of the actions/checkout action and why do most workflows start with it?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Workflow Fundamentals](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Workflow_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The actions/checkout action in GitHub Actions serves a critical purpose by allowing the workflow to access the repository code that's being built or tested. Most workflows start with this action because it ensures that the working directory is properly set up and contains the latest version of the source code.

To understand why, let's break down what actions/checkout does and when you might use it:

Purpose: The actions/checkout action checks out (downloads) the repository into a specified workspace within the GitHub Actions environment. This step is essential because:

It ensures that the workflow runs with the most recent code from the repository.
It allows for consistent and reproducible builds, as it always starts with the latest source code.

Why Most Workflows Start With It:

#### Cons
istency: By using actions/checkout, you guarantee that all steps in your workflow are executed against the same version of the codebase.
Dependency Management: If your build or test scripts depend on specific files or configurations within the repository, having the latest version ensures these dependencies are met accurately.
Environment Setup: The action sets up a clean and isolated environment for each job to run in, ensuring that no local changes interfere with the workflow execution.

#### Example
Workflow:

YAML
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3  # Check out the repository code
        with:
          fetch-depth: 2            # Fetch recent history to support Git LFS files

      - name: Run tests
        run: pytest

#### Common Mistakes

Omitting actions/checkout and running a workflow without ensuring the source code is correctly checked out can lead to discrepancies in results.
Forgetting to specify fetch-depth if large files or complex Git history are involved, which might result in slower workflows.

By always starting with actions/checkout, you ensure that your workflow operates on the latest and most reliable version of the code. This practice is fundamental for maintaining consistency and reliability across CI/CD pipelines.

**Code:**

```yaml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3  # Check out the repository code
        with:
          fetch-depth: 2            # Fetch recent history to support Git LFS files

      - name: Run tests
        run: pytest
```

**Flow:**

1. Start Workflow
2. Use actions/checkout@v3 tocheck out code
3. Ensure consistent and latestsource code for all steps
4. RUN tests or other build tasks

