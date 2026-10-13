---
type: "practice"
difficulty: "Deep"
---

# A path-filtered CI skipped retraining a model whose imported preprocessing library changed, because the model's own paths didn't. How do you make the change detection dependency-aware so it never skips a retrain it should have run?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Monorepo CI Optimization](<../../../../TOOLS/Git_and_GitHub/Git_Core/Monorepo_CI_Optimization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of path-filtered CI skipping retraining when an imported preprocessing library changes, we need to ensure that our Git and GitHub configuration is dependency-aware. This means detecting changes in external libraries and making sure they are included in the model's retraining pipeline.

TL;DR

The core solution involves configuring Git submodules or Git LFS (Large File Storage) for managing dependencies, and setting up a CI/CD pipeline to monitor these dependencies. By doing so, we ensure that any changes in external libraries trigger a retrain, even if the model's own paths do not.

#### Triage

The question is probing how to configure Git and GitHub to detect changes in imported libraries and include them in our CI/CD process. The key assumptions are:

The preprocessing library is stored as an external dependency.
We use a path filter for CI, which skips certain directories or files.
We need the CI system to retrain the model when dependencies change.

#### First Principles Solution

Identify and Manage Dependencies:

Use Git submodules if your library is hosted on GitHub and you have control over it.
Alternatively, use Git LFS for large file storage or directly commit the dependency's files, depending on its size.

Configure CI/CD to Monitor Dependencies:

Set up a pre-commit hook in your local environment to check for changes in dependencies before committing.
Use GitHub Actions or another CI tool to create workflows that monitor these dependencies.

Integrate Dependency Changes into Retraining Pipeline:

Modify the model's retraining pipeline to include steps that detect and include any updated dependencies.
Ensure that when a dependency change is detected, the retraining process is triggered.

#### Implementation
Steps
1. Use Git Submodules

If you control the preprocessing library repository:

BASH
# Add or update submodule in your main project repo
git submodule add https://github.com/your-username/preprocessing-library.git path/to/submodule

# Update and commit changes
cd path/to/submodule
git pull origin master  # or the relevant branch
cd ..
git add path/to/submodule
git commit -m "Updated preprocessing library"
2. Use Git LFS

For larger dependencies:

BASH
# Initialize Git LFS in your project repository
git lfs install

# Track a file with Git LFS
git lfs track "path/to/large-file"

# Add and commit the tracked file
git add .gitattributes path/to/large-file
git commit -m "Add large file tracking"
3. Configure GitHub Actions

Create or update a github/workflows directory in your project to set up workflows:

YAML
name: Retrain Model with Dependencies

on:
  pull_request:
    branches: [ main ]
  push:
    paths:
      - 'path/to/submodule/**'
      - 'path/to/large-file'

jobs:
  retrain_model:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: '3.x'
    - name: Install dependencies
      run: |
        pip install -r requirements.txt  # Ensure your dependencies are in a requirements file
    - name: Retrain model
      run: |
        # Your retraining script here, e.g., python train.py
4. Monitor and Integrate Changes

Ensure that any changes to the preprocessing library or large files trigger a retrain:

PYTHON
# Example of a Python script to check for dependency updates before training
import os

def check_for_updates():
    # Check if submodule has been updated
    os.system('git submodule foreach --recursive git pull origin master')

    # Check if any tracked file has changed
    return os.path.exists('path/to/large-file')

if not check_for_updates():
    print("No updates detected. Retraining model...")
    # Run retraining script
else:
    print("Dependencies updated. Retraining model...")

# Your actual training code here

#### Closing Summary
and Common Mistakes

By using Git submodules or LFS for dependency management, and configuring a CI/CD pipeline to monitor these dependencies, we ensure that any changes trigger the necessary retraining. Common mistakes include:

Not updating submodules or large files correctly.
Failing to integrate dependency checks into the model's retraining logic.
Overlooking paths in path filters that should be included.

#### Follow-Ups
How do you handle dependencies with multiple versions?
What are some best practices for managing large dependencies in Git?
Can you provide an example of a more complex workflow involving multiple models and shared libraries?

**Code:**

```python
import os

def check_for_updates():
    # Check if submodule has been updated
    os.system('git submodule foreach --recursive git pull origin master')

    # Check if any tracked file has changed
    return os.path.exists('path/to/large-file')

if not check_for_updates():
    print("No updates detected. Retraining model...")
else:
    print("Dependencies updated. Retraining model...")

# Your actual training code here
```

**Flow:**

1. Retraining Script
2. No
3. Yes
4. Check for Updates
5. Are dependencies updated?
6. Retraining model...
7. Retraining model...
8. Retrain model script

