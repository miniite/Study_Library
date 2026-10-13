---
type: "practice"
difficulty: "Intermediate"
---

# Explain the default permissions of GITHUB_TOKEN in your alert-config workflow, why you'd set it read-only at the top level, and how you scope up to write for a single job that pushes a monitoring-stack release tag after the replay gate passes.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Token Permissions](<../../../../TOOLS/Git_and_GitHub/Git_Core/Token_Permissions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In an AIOps context, understanding Git and GitHub permissions is crucial for ensuring both security and efficiency in your alert-config workflow. The GITHUB_TOKEN is a default token that provides access to the GitHub API but has default permissions which are often restrictive by design. In this scenario, we will explain why you might set the GITHUB_TOKEN read-only at the top level and how to scope up to write for a specific job.

The GITHUB_TOKEN defaults to having limited permissions to prevent accidental misuse or unauthorized actions. At the top level, setting it to read-only ensures that your token cannot make changes like pushing code, creating branches, or merging pull requests without explicit permission adjustments. This is important from a security standpoint because if you have multiple workflows running under this token, they could potentially be compromised.

To achieve fine-grained control over permissions, we need to scope the GITHUB_TOKEN up specifically for a job that pushes a monitoring-stack release tag after the replay gate passes. Here’s how you can set it up:

Top-Level Token Setup: At the top level, you would configure the GITHUB_TOKEN with read-only access.
Job-Specific Permission Scoping: For the specific job that requires write permissions to push tags, you will need to explicitly grant those permissions using a separate token or by adjusting the existing GITHUB_TOKEN.

Here’s an example of how this can be implemented:

Top-Level Token Configuration:

YAML
secrets:
  GITHUB_TOKEN:
    # Default read-only access

Job-Specific Token Setup: Create a separate token for the job that needs write permissions.

YAML
jobs:
  - name: push-monitoring-stack-release
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v3
        with:
          token: ${{ secrets.GITHUB_TOKEN_READONLY }}  # Use read-only token for this step

      - name: Replay Gate Passed
        run: echo "Replay gate passed, moving to the next step"

      - name: Push Monitoring Stack Release Tag
        uses: actions/upload-artifact@v3
        with:
          name: monitoring-stack-release-tag
          path: path/to/release/tag  # Assuming you have a release tag file

To implement this in GitHub Actions, you might need to use the env or with blocks to specify the token for different steps.

Job-Specific Token with Write Permissions:
YAML
jobs:
  - name: push-monitoring-stack-release
    runs-on: ubuntu-latest
    permissions:  # Explicitly grant write permissions
      contents: read
      pull-requests: read
      deployments: write

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v3
        with:
          token: ${{ secrets.GITHUB_TOKEN_WITH_WRITE }}  # Use the new token with write access for this step

      - name: Replay Gate Passed
        run: echo "Replay gate passed, moving to the next step"

      - name: Push Monitoring Stack Release Tag
        uses: actions/upload-artifact@v3
        with:
          name: monitoring-stack-release-tag
          path: path/to/release/tag  # Assuming you have a release tag file

By following these steps, you ensure that your token setup is both secure and functional. The read-only top-level configuration prevents accidental changes, while the scoped write access allows specific actions to be performed securely.

