---
type: "practice"
difficulty: "Intermediate"
---

# A job succeeds locally in act but fails on GitHub-hosted runners. What environment differences cause this and how do you reconcile them?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Self-hosted Runners](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Self-hosted_Runners.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a GitHub Actions job succeeds locally but fails on GitHub-hosted runners, it often points to subtle differences in the local development environment versus the hosted runner's environment. To understand and resolve this issue, we need to identify key differences, check common configuration issues, and ensure consistent setup.

#### Triage

The question is probing for an understanding of the environment variability between local machines and GitHub-hosted runners. The core idea here is to identify potential discrepancies that could cause a job to fail in one environment but succeed in another. Common factors include different versions of tools, dependencies, network settings, and permissions.

Step-by-Step Solution
Environment Variables: Ensure that all necessary environment variables are correctly set both locally and on GitHub runners.
Tool Versions: Check the versions of tools like Node.js, Python, or any other dependencies used in the job. Sometimes, a local development setup might use an outdated version of a tool that is incompatible with newer ones.
Dependencies: Verify that all required dependencies are installed locally and are correctly listed in your package.json, requirements.txt, etc., files. Ensure that these dependencies match those on the GitHub runners.
Network Configuration: Differences in network connectivity or firewall rules can lead to failure, especially if a job relies on external services.
Permissions: Permissions might differ between local and remote environments, affecting file access or repository interactions.
Detailed Steps
Check Logs:
Review the logs from both successful (local) and failing (GitHub runner) runs to identify any errors or warnings.
Dependency Versions:
Run npm ls or pip freeze > requirements.txt locally to list installed packages and compare with your GitHub Actions workflow file.
Environment Variable Consistency:
Ensure that environment variables like GITHUB_TOKEN or custom-defined variables are correctly set in both environments.
Tool Version Management:
Use a tool like nvm for Node.js to manage different versions locally and ensure the same version is used in GitHub Actions via actions/setup-node@v2.
Network Diagnostics:
Check if any network services required by your job are accessible from the GitHub runners.

#### Example
Workflow

Here’s a sample YAML snippet that demonstrates how you might configure these checks:

YAML
name: CI

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '14'

      - name: Install dependencies
        run: npm ci

      - name: Check environment variables
        run: echo ${{ secrets.GITHUB_TOKEN }}

      - name: Run tests
        run: npm test

  # Add additional steps as needed to check other aspects of the environment

#### Common Mistakes
and Follow-Ups
Failing to Compare Local and Remote Environments: Often, developers overlook this step, leading to subtle differences.
Insufficient Logging: Not having adequate logging can make it hard to pinpoint the issue.
Overlooking Network Issues: External services might behave differently depending on network conditions.

By systematically checking these areas, you can identify and resolve the discrepancies that cause your GitHub Actions job to fail locally but succeed on GitHub-hosted runners. This ensures consistent behavior across different environments.

**Code:**

```yaml
name: CI

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '14'

      - name: Install dependencies
        run: npm ci

      - name: Check environment variables
        run: echo ${{ secrets.GITHUB_TOKEN }}

      - name: Run tests
        run: npm test
```

