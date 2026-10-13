---
type: "practice"
difficulty: "Deep"
---

# How do you set least-privilege permissions on GITHUB_TOKEN across a repo and why is the default read/write a liability?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ GITHUB_TOKEN Permissions](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/GITHUB_TOKEN_Permissions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with GitHub Actions, setting up the correct permissions for GITHUB_TOKEN is crucial to ensure that your workflows are both secure and perform their intended tasks efficiently. By default, GITHUB_TOKEN has read/write access across the repository, which can be a significant security risk if not properly managed.

#### Triage
Understanding the Question

The question probes how you manage permissions for tokens in GitHub Actions to minimize security risks while ensuring that actions have sufficient permissions to execute their tasks. It also highlights the importance of understanding default permissions and why they might pose liabilities.

Triaging Assumptions and First Principles
Default Permission Level: The default GITHUB_TOKEN has read/write access to the repository.
Security Risk: This default permission level can be a security vulnerability if misused or exposed.
Least Privilege Principle: It's best practice to grant the minimum necessary permissions to tokens for security reasons.

#### Solution
Least Privilege Permissions for GITHUB_TOKEN

To set up least privilege permissions on GITHUB_TOKEN in GitHub Actions, follow these steps:

Identify the Minimal Required Permissions:

Determine which specific actions need read access and which require write access.
For example, a workflow that pushes code to a branch might only require read access to certain files, but writing to commit changes.

Create Custom GitHub Tokens:

Go to your repository's settings in the GitHub UI.
Navigate to "Secrets & variables" -> "New repository secret."
Create multiple tokens with specific permissions, such as read_only_token and write_only_token.

Configure Workflow Files:

In your .github/workflows directory, create or update YAML files for your workflows.
Use the appropriate token in each workflow file based on its required access level.

Here’s an example of a GitHub Actions workflow:

YAML
name: Example Workflow

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Use read-only token for reading files
        env:
          GITHUB_TOKEN: ${{ secrets.read_only_token }}
        run: |
          echo "Reading file content..."
          cat path/to/read/file.txt

      - name: Use write-only token for committing changes
        env:
          GITHUB_TOKEN: ${{ secrets.write_only_token }}
        run: |
          echo "Committing and pushing changes..."
          git config user.name 'GitHub Action'
          git config user.email 'github-action@example.com'
          git add path/to/commit/file.txt
          git commit -m "Automated commit"
          git push origin main

#### Closing Summary
+ Common Mistakes/Follow-Ups

By implementing least privilege permissions, you ensure that GitHub Actions have the minimum necessary access to perform their tasks. This reduces potential security risks associated with default read/write tokens.

#### Common Mistakes

Not rotating or revoking unused tokens.
Using a single token for multiple workflows without proper segmentation of permissions.
Failing to monitor and audit token usage over time.

#### Follow-Ups

How often should the tokens be rotated?
What monitoring tools can help in auditing token usage?
How do you handle multi-repository workflows?

**Code:**

```yaml
name: Example Workflow

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Use read-only token for reading files
        env:
          GITHUB_TOKEN: ${{ secrets.read_only_token }}
        run: |
          echo "Reading file content..."
          cat path/to/read/file.txt

      - name: Use write-only token for committing changes
        env:
          GITHUB_TOKEN: ${{ secrets.write_only_token }}
        run: |
          echo "Committing and pushing changes..."
          git config user.name 'GitHub Action'
          git config user.email 'github-action@example.com'
          git add path/to/commit/file.txt
          git commit -m "Automated commit"
          git push origin main
```

**Flow:**

1. Checkout Repository
2. Use read-only token forreading files
3. Read file content...
4. Use write-only token forcommitting changes
5. Commit and push changes...

