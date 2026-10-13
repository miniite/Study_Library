---
type: "practice"
difficulty: "Expert"
---

# You're building a bot that auto-labels and routes pipeline PRs by changed dataset path and enforces data-owner review on the pii datasets directory. Compare a GitHub App, Actions, and webhooks for this, focusing on rate limits, permission scoping, and what happens when the creating user leaves.

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Secure Workflows for Forks](<../../../../TOOLS/Git_and_GitHub/Git_Core/Secure_Workflows_for_Forks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build an auto-labeling and routing bot for GitHub Pull Requests (PRs) based on changes in dataset paths, with specific attention to rate limits, permission scoping, and handling users who leave, we need to consider three main approaches: a GitHub App, Actions, and webhooks. Each has its own strengths and limitations, which will affect the overall implementation.

TL;DR

GitHub Apps offer more fine-grained control over permissions and can leverage API rate limits effectively. GitHub Actions provide a robust workflow for automation but might face rate limit issues if not managed carefully. Webhooks are simpler to set up but lack some of the advanced features available in the other two options. The choice depends on the specific requirements, especially regarding rate limits and permission management.

#### Triage

The question is asking us to evaluate three different GitHub-based tools for automating PR processes: Apps, Actions, and webhooks. We need to consider:

Rate Limits: How each method handles API calls and potential bottlenecks.
Permission Scoping: What level of access is required for the bot, and how this affects the overall security and compliance.
User Departure Handling: What happens if the user who created a PR leaves the organization.

#### First-Principles Solution
GitHub App

Rate Limits: A GitHub App can use the metadata scope to limit API usage specifically for app tokens, which might be beneficial in controlling rate limits more effectively. However, you need to manage the installation of the app by each repository owner, which adds some complexity.

Permission Scoping: The bot can request read and write permissions scoped to specific repositories or even directories like the pii datasets directory. This ensures that sensitive data handling is controlled and compliant with security policies.

User Departure Handling: If a user leaves the organization, their access tokens will become invalid. GitHub Apps have mechanisms to handle this by revoking app installation permissions for users who no longer work in relevant teams or repositories.

GitHub Actions

Rate Limits: GitHub Actions operate under the same rate limits as regular API calls. However, they can be configured to run on specific events (e.g., push, pull_request) and use caching mechanisms to mitigate frequent API calls.

Permission Scoping: Similar to Apps, Actions can also request scoped permissions. For example, you could set up an Action that checks changes in the pii directory and enforces a review from the data owner.

User Departure Handling: When a user leaves, their access tokens will expire or be revoked. This means any GitHub Actions triggered by PRs created by this user might fail until they are reinstalled or new permissions are granted.

Webhooks

Rate Limits: Webhooks do not have built-in rate limit handling; however, you can implement custom rate limiting in your application backend to manage API calls more effectively.

Permission Scoping: Webhooks require the repository owner to install and maintain them. The bot will need read and write permissions to trigger actions like labeling PRs or requesting reviews. However, managing these webhooks for multiple repositories manually can be cumbersome.

User Departure Handling: Unlike Apps and Actions, webhooks do not have built-in mechanisms to handle user departures effectively. If a user leaves, you would need to manually update the webhook configuration in each repository where it is installed.

#### Closing Summary

Each method has its strengths and weaknesses when building an auto-labeling and routing bot for GitHub PRs. GitHub Apps provide better control over rate limits and permission scoping but require managing app installations. Actions offer a flexible automation framework but face similar rate limit issues without advanced configuration. Webhooks are simpler to set up but lack some of the advanced features available in the other two options.

#### Common Mistakes/Follow-Ups
Overlooking Rate Limits: Ensure that your implementation accounts for potential API rate limits, especially if frequent API calls are required.
Permission Management: Carefully scope permissions to avoid security risks while ensuring the bot can perform necessary actions.
Handling User Departure: Implement a mechanism or process to handle user departures to avoid stale access tokens and failed automation processes.

**Comparison:**

```
Method	Rate Limits	Permission Scoping	User Departure Handling
GitHub App	Effective	Fine-grained	Revocation mechanisms
Actions	Moderate	Fine-grained	Manual revocation
Webhooks	Basic	Limited	No built-in handling
```

**Code:**

```python
# Example of a GitHub Action in YAML
name: Auto-label PR

on:
  pull_request:

permissions:
  contents: read
  pull-requests: write

jobs:
  label-pr:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Auto-label PR
        run: |
          # Code to auto-label PR based on changes in dataset paths and pii directory
```

**Example input / output:**

```text
Input:
- A PR is created with changes in the pii dataset path.
- The user who created the PR leaves the organization.

Output:
1. GitHub App installs are updated to remove access for the departed user.
2. Actions automatically fail due to missing permissions, triggering a warning or alert.
3. Webhooks continue to trigger but might encounter rate limit issues if not properly managed.
```

**Data:**

```json
{
  "repositories": [
    {
      "name": "data-processing",
      "permissions": {
        "contents": "read",
        "pull-requests": "write"
      }
    },
    {
      "name": "analytics-tools",
      "permissions": {
        "contents": "read",
        "pull-requests": "write"
      }
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. No
4. Yes
5. PR Created
6. User Leaves?
7. App Revokes Access
8. Actions Continue
9. Notify Admins
10. Rate Limit Alerts
11. Webhook Continues?
12. Disable Webhook
13. Monitor Rate Limits

