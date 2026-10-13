---
type: "practice"
difficulty: "Deep"
---

# A reusable training workflow shared across all model repos was updated and silently broke retrains in repos pinned to @main. How do you version reusable workflows so a central change can't break pinned consumers, and roll a fix org-wide?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reusable Workflows](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reusable_Workflows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When faced with the challenge of maintaining a reusable training workflow shared across multiple model repositories, it’s crucial to ensure that any changes made in the central repository do not inadvertently break pinned consumers. The primary goal here is to implement robust versioning and change management strategies that allow for smooth updates while minimizing the risk of breaking existing deployments.

The question essentially probes your ability to design a versioned workflow that can be updated without affecting downstream dependencies, as well as how you would roll out these changes in an organization-wide manner. This involves understanding Git/GitHub workflows, version control best practices, and strategic planning.

#### Triaging the Question
What is being asked? The key challenge here is ensuring that a central training workflow can be updated without breaking consumers who are pinned to specific branches or tags.

#### Assumptions
We assume the training workflow is implemented using Git/GitHub, with multiple repositories consuming this workflow. Consumers might have their own main branch or other branches tagged for deployment.
Steps involved:
Design a versioned system for workflows.
Implement a branching strategy to manage central changes.
Ensure consumers can opt into new versions without breaking existing deployments.

#### Solution
from First Principles
Step 1: Versioning the Workflow

To avoid breaking existing deployments, we need to introduce a clear and consistent versioning scheme for our workflow files. This could be done using semantic versioning or a custom scheme that fits the needs of the organization.

BASH
# Example directory structure for versioned workflows
/workflows/
    v1-0-0/  # Initial release
    v1-1-0/  # First update
    ...

Each version should contain all the necessary changes and configuration files, ensuring that consumers can simply switch to a new version without needing significant reconfiguration.

Step 2: Branching Strategy for Central Repository

Implement a branching strategy where central updates are made on a dedicated branch. For example, create a develop branch as a staging area for new features or fixes before merging into the mainline (main).

BASH
# Example Git flow for updating workflows
git checkout develop  # Switch to the development branch
# Make changes and commit them
git push origin develop

# Once tested, merge develop into main
git checkout main
git merge develop -m "Merge update"
git push origin main
Step 3: Consumer Branch Management

#### Cons
umers should be on specific branches that are pinned to the desired workflow version. If a new version is needed, consumers can either be updated manually or through automated scripts.

BASH
# Example branch for consumer A using v1-0-0
git checkout -b consumer-A-v1-0-0
git pull origin consumer-A-v1-0-0  # Ensure it's up to date

# To update to the latest version, change the branch
git checkout -b consumer-A-v1-1-0
git pull origin consumer-A-v1-1-0
Step 4: Centralized Rollout Mechanism

To roll out changes organization-wide, we can use a combination of manual and automated processes. For example, a CI/CD pipeline could be set up to notify all consumers about new workflow versions.

YAML
# Example GitHub Actions for notifying consumers
name: Notify Consumers

on:
  push:
    branches:
      - develop
  pull_request:
    branches:
      - develop

jobs:
  notify_consumers:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Notify Slack
        if: github.ref == 'refs/heads/develop'
        run: |
          curl -X POST https://slack.com/api/chat.postMessage \
            -H "Authorization: Bearer ${{ secrets.SLACK_BOT_TOKEN }}" \
            -d "New workflow version v1-1-0 is available. Consumers should update if necessary."

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the key to solving this problem lies in a well-defined versioning strategy, strategic branching for central updates, and automated mechanisms for notifying consumers. By implementing these steps, we can ensure that any changes made to the training workflow do not break existing deployments.

#### Common mistakes

Failing to maintain proper version control of shared workflows.
Not having a clear branching strategy to manage changes.
Inadequate communication or notification systems for rolling out updates.

#### Follow-ups

How would you handle edge cases where consumers are on an old branch?
What tools or scripts would you use to automate the update process?
How would you ensure that all consumers have adopted new versions?

**Comparison:**

| Workflow Version | Branch Name       | Description                              |
|------------------|-------------------|------------------------------------------|
| v1-0-0           | consumer-A-v1-0-0  | Initial version for consumer A            |
| v1-1-0           | consumer-A-v1-1-0  | Updated version for consumer A            |

**Code:**

```bash
# Example Git command for checking out a specific versioned workflow
git checkout v1-0-0

# Example GitHub Actions configuration for notifying Slack
name: Notify Consumers

on:
  push:
    branches:
      - develop
  pull_request:
    branches:
      - develop

jobs:
  notify_consumers:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Notify Slack
        if: github.ref == 'refs/heads/develop'
        run: |
          curl -X POST https://slack.com/api/chat.postMessage \
            -H "Authorization: Bearer ${{ secrets.SLACK_BOT_TOKEN }}" \
            -d "New workflow version v1-1-0 is available. Consumers should update if necessary."
```

**Example input / output:**

```text
# Example input for notifying Slack about a new version
curl -X POST https://slack.com/api/chat.postMessage \
  -H "Authorization: Bearer ${{ secrets.SLACK_BOT_TOKEN }}" \
  -d "New workflow version v1-1-0 is available. Consumers should update if necessary."
```

**Data:**

```yaml
# Example Slack message configuration for GitHub Actions
name: Notify Consumers

on:
  push:
    branches:
      - develop
  pull_request:
    branches:
      - develop

jobs:
  notify_consumers:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Notify Slack
        if: github.ref == 'refs/heads/develop'
        run: |
          curl -X POST https://slack.com/api/chat.postMessage \
            -H "Authorization: Bearer ${{ secrets.SLACK_BOT_TOKEN }}" \
            -d "New workflow version v1-1-0 is available. Consumers should update if necessary."
```

**Flow:**

1. Central Workflow Repository
2. Develop Branch
3. Merge into Main
4. Consumer Repositories
5. Consume from v1-0-0
6. Notify Slack on New Version
7. Update to v1-1-0 if Necessary

