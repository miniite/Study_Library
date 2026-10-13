---
type: "practice"
difficulty: "Deep"
---

# How do you integrate CodeQL or Trivy scanning into a pipeline without blocking every PR on pre-existing findings?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reusable Workflows](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Reusable_Workflows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To integrate CodeQL or Trivy scanning into your GitHub Actions workflow, the key is to strike a balance between thoroughness and flexibility, ensuring that new vulnerabilities are caught while existing ones don't halt every pull request (PR). This involves setting up actions that can handle both static analysis and dynamic testing without blocking PRs on pre-existing findings. Here’s how you can achieve this:

First, let's quickly summarize the approach: We will use GitHub Actions to run CodeQL or Trivy scans as part of our CI/CD pipeline but configure it in such a way that it only fails when new issues are introduced by recent changes. Existing vulnerabilities from previous runs won’t block PRs immediately.

**Comparison:**

```
Step	Action	Purpose
1	Checkout Repo	Fetches the latest code changes
2	Setup Python	Ensures dependencies are installed
3	Install Dependencies	Installs Trivy for scanning
4	Scan Code	Runs vulnerability scan
5	Report Results	Logs findings if new issues are present
```

**Code:**

```yaml
name: Vulnerability Scan

on:
  pull_request:
    branches:
      - main   # Specify the branch to which this workflow applies

jobs:
  codeql_scan:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout Repository
      uses: actions/checkout@v3

    - name: Setup Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.8'   # Adjust based on your project's requirements

    - name: Install dependencies
      run: pip install trivy

    - name: Scan code for vulnerabilities
      id: scan
      run: |
        trivy image --exit-code 1 your-image-name   # Use CodeQL if your images are built from a Git repo

    - name: Report results
      if: steps.scan.outputs.exitCode != 0
      run: echo "Vulnerabilities found. Reviewing PR." >> $GITHUB_OUTPUT
```

**Example input / output:**

```text
Given a new PR to `main` branch, this workflow would check for vulnerabilities in the image used by your application. If any are found due to changes introduced in the PR, it will log an issue and proceed with the merge. However, if existing issues persist without any change in the PR, no blockage occurs.
```

**Data:**

```yaml
# Example of the GitHub Actions configuration above.
```

**Flow:**

1. Yes
2. No
3. Yes
4. GitHub Actions triggered
5. Is PR on main?
6. Checkout Repository
7. Setup Python Environment
8. Install Trivy Dependencies
9. Run CodeQL or Trivy Scan
10. Is Exit Code 0?
11. Log Vulnerabilities Found
12. Continue PR Merge
13. Pass CI/CD Check

