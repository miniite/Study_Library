---
type: "practice"
difficulty: "Intermediate"
---

# Your GitHub webhook to the GitOps controller fires on every push including runbook-doc-only commits, causing needless reconciles and rate-limit noise on the monitoring cluster. How do you filter webhook events by path or type so the controller only syncs on real rule or pipeline changes?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reducing Actions Costs](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reducing_Actions_Costs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of unnecessary webhooks firing due to runbook-doc-only commits, we need to implement a filtering mechanism in our GitHub webhook setup that distinguishes between relevant and non-relevant push events. This involves using GitHub Actions within the repository to filter out specific paths or commit types before notifying the GitOps controller.

#### Triage
What's Being Probed?

The interviewer is probing your understanding of:

GitHub Webhook Mechanics: How webhooks work in triggering actions based on various events.
Filtering Mechanisms: Techniques for filtering webhook payloads to ensure only relevant events are processed.
GitOps Practices: Best practices for ensuring efficient and reliable GitOps workflows.

#### Assumptions
The repository contains both runbooks (docs) and actual configuration files that need to be synced with the monitoring cluster.
The GitOps controller is set up to process specific types of changes, such as those related to pipelines or rules.

#### Solution
Implementing Path-Based Filtering in GitHub Actions

Create a GitHub Action We will create a GitHub Action that runs on push events and filters out commits based on their paths.

Path Matching Rules Define the path patterns for runbook documents and configuration files. For instance:

Runbooks: /docs/*
Configuration Files: /config/*

Action Workflow The GitHub Action will process the commit message and check if it matches any of the defined paths. If not, it won't notify the GitOps controller.

Step-by-Step Implementation
Create a github/workflows directory in your repository.
Add an .yaml file for the GitHub Action, e.g., filter-webhook-events.yaml.
YAML
name: Filter Webhook Events
on:
  push:
    branches:
      - main # or any other branch you are working on

jobs:
  filter-commit:
    runs-on: ubuntu-latest
    steps:
    - name: Check Commit Paths
      run: |
        COMMIT_MESSAGE=$(git log --format=%B -n 1 HEAD)
        if ! echo "$COMMIT_MESSAGE" | grep -qE '/docs/|/config/'; then
          echo "No action required for this commit."
          exit 0
        fi
    - name: Notify GitOps Controller
      if: always()
      run: |
        echo "Commit message contains changes to important paths. Notifying GitOps controller..."
        # Here you can add logic to send a notification or trigger an event, e.g., with curl or another tool.

#### Closing Summary
and Common Mistakes/Follow-Ups

By implementing this path-based filtering mechanism, we ensure that the GitOps controller only receives notifications for commits related to critical configurations. This reduces unnecessary reconciles, improves performance, and avoids rate-limit issues.

#### Common mistakes
include:

Not defining clear path patterns.
Failing to properly check commit messages or paths.
Overlooking edge cases where the commit message might not contain paths explicitly.

For follow-ups, you could explore more advanced filtering techniques using Git hooks or external tools like git rev-list and git diff.

**Code:**

```yaml
name: Filter Webhook Events
on:
  push:
    branches:
      - main

jobs:
  filter-commit:
    runs-on: ubuntu-latest
    steps:
    - name: Check Commit Paths
      run: |
        COMMIT_MESSAGE=$(git log --format=%B -n 1 HEAD)
        if ! echo "$COMMIT_MESSAGE" | grep -qE '/docs/|/config/'; then
          echo "No action required for this commit."
          exit 0
        fi
    - name: Notify GitOps Controller
      if: always()
      run: |
        echo "Commit message contains changes to important paths. Notifying GitOps controller..."
```

**Example input / output:**

```

No specific input/output is required for this solution, but here’s a dry-run example:

SH
Copy
$ git log --format=%B -n 1 HEAD
Commit message: "Update runbook documentation"
$ echo $COMMIT_MESSAGE | grep -qE '/docs/|/config/'
# Returns false since the commit doesn't contain paths related to config.
No action required for this commit.
```

**Flow:**

1. Match
2. No Match
3. GitHub Push Event
4. Check Commit Paths
5. Notify GitOps Controller
6. Do Nothing

