---
type: "practice"
difficulty: "Intermediate"
---

# You want a pre-commit hook that blocks committing a notebook with non-empty output cells (which have leaked log samples before) or any file over 50 MB in the telemetry-analysis repo, but a teammate committed via a client without the hook. Why can't client hooks be a real gate, and how do you enforce it in CI?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Server-side Hooks and Push Protection](<../../../../TOOLS/Git_and_GitHub/Git_Core/Server-side_Hooks_and_Push_Protection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In an AIOps environment, maintaining data quality is crucial, especially when dealing with sensitive information like logs and large telemetry files. To address this, we want to prevent committing notebooks with non-empty output cells or any file over 50 MB in the telemetry-analysis repo via a pre-commit hook. However, if a teammate committed directly from a client without the hook, it's important to understand why client hooks can't serve as a reliable gate and how we can enforce this policy through CI.

Client hooks are useful but not foolproof because they depend on the client environment where they run. A teammate might have forgotten to install or configure the pre-commit hook locally, or their local Git configuration might differ from what's expected in the repository settings. Additionally, client hooks don't provide the same level of consistency and visibility as CI systems do.

To enforce these rules effectively, we need a comprehensive CI pipeline that runs before merging into the main branch. Here’s how you can set this up:

Understand Why Client Hooks Aren’t Reliable:

Client hooks run locally on the developer's machine.
They rely on the local Git configuration and environment.
If a developer forgets to install or configure them, they won't be enforced.

Set Up CI Hooks:

Use GitHub Actions (or another CI/CD platform) to define these checks.
Create a ci.yml file in the repository that defines jobs to run pre-merge checks.

Pre-Merge Checks via CI:

Define steps in your CI configuration to check for non-empty output cells and large files.
For notebooks, you can use tools like Jupyter Notebooks' APIs or custom scripts to inspect cell contents.
Use git lfs (Large File Storage) to manage file sizes effectively.

#### Example
GitHub Actions Workflow:

YAML
name: Pre-Merge Checks

on:
  pull_request:

jobs:
  pre-merge-checks:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: '3.x'

    - name: Install dependencies
      run: |
        pip install jupyter
        pip install git-lfs

    - name: Check notebook output cells
      run: |
        # Example script to check for non-empty output cells in notebooks
        echo "Checking for non-empty output cells..."
        python check_notebooks.py

    - name: Check file sizes
      run: |
        # Example command to check file sizes using git lfs
        git lfs ls-files --exclude-filter-b 'pattern/exclude_large_files.txt'

Handling Large Files with Git LFS:

Use git lfs to track large files.
Configure .gitattributes to mark these files for Git LFS:
PLAINTEXT
*.{csv,log,png} filter=lfs diff=lfs merge=lfs -text
Ensure this configuration is applied consistently across all contributors.

#### Common Mistakes
and Follow-Ups:

Forgetting to include the git lfs setup in the repository.
Not properly configuring the .gitattributes file.
Assuming client hooks are sufficient without complementary CI checks.
Ignoring edge cases, such as developers committing large files outside of notebooks.

By setting up robust CI pipelines that enforce these rules, you can ensure data quality and security while maintaining a developer-friendly workflow.

