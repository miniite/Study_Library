---
type: "practice"
difficulty: "Deep"
---

# What is the concept of workflow permissions and how do you set the permissions of GITHUB_TOKEN?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ GITHUB_TOKEN Permissions](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/GITHUB_TOKEN_Permissions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Workflow permissions in GitHub Actions are a crucial aspect that governs which actions can be performed within a workflow file, effectively controlling access to resources like repositories and secrets. Essentially, these permissions dictate what the GITHUB_TOKEN is allowed to do when running jobs. To set the permissions of GITHUB_TOKEN, you need to understand both the default behavior and how to customize it for specific workflows.

#### Triage

The question asks about workflow permissions in GitHub Actions and specifically focuses on setting up the GITHUB_TOKEN. The key points here are:

Understanding Default Permissions: By default, a GITHUB_TOKEN has read access to repositories.
Customizing Access Levels: You can customize the access levels of the GITHUB_TOKEN depending on your needs.

#### First-Principles Solution

Let's break down how permissions work and how you can set them up:

Default Behavior

By default, a GITHUB_TOKEN created from a repository has read-only access to that repository. This is done for security reasons, ensuring that automated actions do not have full control over the repository without explicit permission.

Customizing Access Levels

To customize the access levels of the GITHUB_TOKEN, you can use the permissions field in your workflow file. Here’s how:

Identify the Token: First, ensure you are using a specific token or that the workflow is configured to use an environment variable for the GITHUB_TOKEN.
Configure Permissions: Use the permissions key in the job configuration to specify which actions can be performed.

Here's an example of how to set up workflow permissions:

YAML
name: Example Workflow

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read              # Read access to repository files.
      id-token: write             # Write access to create and use an ID token (optional).
      issues: none                # No access to manage issues.
      pull-requests: none         # No access to manage pull requests.
      statuses: read               # Read-only access to the GitHub API for status checks.

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Run a simple script
        run: echo "Hello, World!"

In this example, we've configured permissions as follows:

contents: Set to read access.
id-token: Write access (if needed).
issues, pull-requests, and statuses: Set to none.

#### Closing Summary

Setting up the correct permissions for a GITHUB_TOKEN in GitHub Actions is essential for maintaining security while allowing necessary actions. By carefully configuring these permissions, you can ensure that your workflows have just enough access to accomplish their tasks without exposing sensitive information or unnecessary risks.

#### Common Mistakes
and Follow-Ups
Default Read Access: Failing to understand the default read-only access of tokens.
Overly Permissive Permissions: Granting more permissions than necessary, which increases security risks.
Misconfiguration: Forgetting to include specific permission keys in the workflow file.

Common follow-up questions might include:

How do you ensure secure handling of GITHUB_TOKEN?
What are some best practices for managing tokens and secrets?
Can you discuss a scenario where overly restrictive permissions could hinder a necessary workflow?

**Code:**

```yaml
name: Example Workflow

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read              # Read access to repository files.
      id-token: write             # Write access to create and use an ID token (optional).
      issues: none                # No access to manage issues.
      pull-requests: none         # No access to manage pull requests.
      statuses: read               # Read-only access to the GitHub API for status checks.

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Run a simple script
        run: echo "Hello, World!"
```

