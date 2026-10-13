---
type: "practice"
difficulty: "Deep"
---

# You need to trigger a downstream workflow in another repo. Why does GITHUB_TOKEN not trigger it and what are your safe alternatives?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ GITHUB_TOKEN Permissions](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/GITHUB_TOKEN_Permissions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of triggering a downstream GitHub Actions workflow from one repository, we first need to understand why GITHUB_TOKEN alone is insufficient. The GITHUB_TOKEN in the context of GitHub Actions provides authentication for the current repository and does not have permissions by default to trigger workflows in other repositories.

TL;DR:

Why GITHUB_TOKEN doesn't work: It lacks permission to access external repositories.
Safe alternatives: Use a personal access token (PAT) or a custom service account with appropriate scopes.

#### Triaging the Question

The question is probing your understanding of GitHub Actions' security and permissions model. We need to explore why GITHUB_TOKEN cannot be used for cross-repository triggers, discuss potential risks associated with this limitation, and identify viable alternatives that maintain security and compliance standards.

Step-by-Step Solution from First Principles

Understanding GITHUB_TOKEN:

The GITHUB_TOKEN generated in a repository is meant to authenticate actions within the same repository.
It does not have permission by default to interact with other repositories, even if they are owned by the same user or organization.

Permissions and Security Concerns:

Cross-repository triggers could expose sensitive information and increase security risks unless properly managed.
If a GITHUB_TOKEN from one repository had access to another, it would introduce a significant security vulnerability.

Using Personal Access Tokens (PAT):

A PAT is a dedicated token that can be granted specific permissions across repositories.
You can use this token in your GitHub Actions workflow to trigger jobs in other repositories securely.

Setting Up PAT for Cross-Repository Triggers:

Generate a PAT with the necessary scopes from the organization's settings or repository secrets.
Store and reference this token within your workflow as an environment variable.

#### Example
GitHub Actions Workflow Using PAT:

YAML
name: Trigger Downstream Job

on:
  push:
    branches:
      - main

jobs:
  trigger-downstream-job:
    runs-on: ubuntu-latest
    env:
      DOWNSTREAM_GITHUB_TOKEN: ${{ secrets.DOWNSTREAM_GITHUB_TOKEN }}
    steps:
      - name: Trigger Downstream Job
        run: |
          echo ::set-output name=token::${{ env.DOWNSTREAM_GITHUB_TOKEN }}
          curl -X POST https://api.github.com/repos/${{ secrets.REPO_NAME }}/actions/workflows/${{ secrets.DOWNSTREAM_JOB_ID }}/dispatches -H "Authorization: token ${{ env.DOWNSTREAM_GITHUB_TOKEN }}" -d '{"ref": "main"}'

Using Custom Service Accounts:

Another approach is to create a dedicated service account with limited permissions, specifically for the purpose of triggering workflows in other repositories.
This service account can be used instead of a PAT and provides finer control over permissions.

Security Best Practices:

Regularly review and rotate tokens to minimize risk exposure.
Keep tokens securely stored and avoid hardcoding them in your repository.
Implement rate limits and monitoring to detect unauthorized use or abuse.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, while GITHUB_TOKEN is sufficient for authentication within a single repository, it does not have the necessary permissions to trigger workflows in other repositories. To safely achieve this, consider using a personal access token with appropriate scopes or a custom service account. Both approaches require careful management and adherence to security best practices.

#### Common mistakes
include:

Using GITHUB_TOKEN directly for cross-repository triggers.
Hardcoding sensitive tokens within the workflow files.
Failing to implement adequate security controls such as rate limits and monitoring.

For follow-ups, you might be asked about specific implementation details or how these solutions integrate with existing workflows. Ensure your responses are clear and aligned with best practices in secure software development.

