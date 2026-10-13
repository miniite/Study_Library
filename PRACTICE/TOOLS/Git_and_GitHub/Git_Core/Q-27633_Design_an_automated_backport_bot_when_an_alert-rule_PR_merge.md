---
type: "practice"
difficulty: "Deep"
---

# Design an automated backport bot: when an alert-rule PR merges to main with a backport/stable label, it cherry-picks to the release branch and opens a PR. What conflict, ordering, and rule-dependency issues must it handle so it never silently drops a suppression?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Automated Dependency Updates](<../../../../TOOLS/Git_and_GitHub/Git_Core/Automated_Dependency_Updates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an automated backport bot that ensures no suppression rules are silently dropped, we need to address several key aspects: conflict handling, PR ordering, and rule dependency management. The core idea is to ensure that every change in main that requires a backport is correctly identified, cherry-picked, and propagated to the appropriate release branches without missing any critical updates.

The bot will:

Monitor for PRs with specific labels (e.g., backport/stable) on merges to the main branch.
Automatically cherry-pick changes from these PRs onto the relevant release branches.
Ensure that any conflicts are resolved before opening a new PR in the target branch.
Handle dependencies between rule suppressions and other PRs.

Let’s break down each step in detail:

#### Triage
What is Being Probed

The interviewer is likely probing your understanding of:

Git workflows and operations: Cherry-picking, merges, and conflict resolution.
Branch management and automation: How to efficiently manage multiple branches with automated tools.
Rule dependency handling: Ensuring that rule suppressions do not get dropped inadvertently.

#### Assumptions
The repository follows a standard Git branching model where main is the development branch, and release branches like release/vX.Y.Z are used for maintenance and bug fixes.
There are predefined labels (e.g., backport/stable) applied to PRs that require backporting.
Conflicts can be resolved manually or automatically using a script.
The bot should handle both simple and complex rule dependency scenarios.

#### First-Principles Solution
Label Monitoring: Monitor the main branch for PR merges with specific labels (backport/stable).
Identify Affected Rules: Extract the suppression rules from the merged PR.
Cherry-Pick Changes: Automate the cherry-picking of changes to the relevant release branches.
Conflict Resolution: Ensure conflicts are identified and resolved before opening a new PR.
Dependency Management: Handle dependencies between rules, ensuring that no suppression is missed.
Step-by-Step Solution

Label Monitoring

Use GitHub Actions or GitLab CI/CD to monitor the main branch for merges with specific labels (e.g., backport/stable).

Extract Affected Rules

Parse the merged PR’s commit messages and files to identify any rule suppressions.
Maintain a list of rules and their dependencies.

Cherry-Pick Changes

Use Git commands or an existing tool like git cherry-pick to apply changes from the main branch to the release branches.
Automate this process using scripts that can handle multiple branches efficiently.

Conflict Resolution

Implement a conflict detection and resolution mechanism.
For simple conflicts, use automated tools.
For complex conflicts, provide manual intervention options.

Dependency Management

Maintain a dependency graph of rules to ensure that no rule is suppressed without its prerequisites being applied first.
Use a version control system (e.g., Git) or an external dependency management tool to track and resolve dependencies.

#### Closing Summary

To summarize, the automated backport bot must handle label monitoring, rule extraction, cherry-picking, conflict resolution, and dependency management. By following these steps, we can ensure that no critical suppressions are missed, maintaining the integrity of our release branches.

#### Common Mistakes/Follow-Ups
Overlooking Conflicts: Failing to properly resolve conflicts can lead to bot failures.
Inadequate Dependency Handling: Missing dependencies between rules can result in suppressed alerts not being applied correctly.
Manual Intervention: Over-relying on manual steps can slow down the process and introduce errors.

**Comparison:**

```
Yes
No
Yes
No
PR Merge
Has 'backport/stable' Label?
Extract Rules
Skip PR
Cherry-Pick to ReleaseBranches
Resolve Conflicts
Check Dependencies?
Apply Prerequisites
Open Backport PRs
```

**Code:**

```python
import os
from github import Github

def get_merged_prs_with_label(repo, label):
    prs = []
    for pull in repo.get_pulls(state='closed'):
        if any(label in l.name for l in pull.labels):
            prs.append(pull)
    return prs

def cherry_pick_changes(pr, release_branch):
    # Implementation of cherry-picking logic
    pass

def resolve_conflicts(cherry_picked_commit, target_branch):
    # Conflict resolution logic
    pass

def main():
    g = Github("your_api_token")
    repo = g.get_repo("user/repo_name")

    prs = get_merged_prs_with_label(repo, "backport/stable")
    for pr in prs:
        release_branches = ["release/v1.0", "release/v2.0"]
        cherry_picked_commits = [commit.sha for commit in pr.get_commits()]
        for branch in release_branches:
            if not cherry_pick_changes(pr, branch):
                resolve_conflicts(cherry_picked_commit, branch)

if __name__ == "__main__":
    main()
```

**Example input / output:**

| PR Number | Action Taken                  | Conflicts | Dependencies |
|-----------|-------------------------------|-----------|--------------|
| 123       | Cherry-picked to release/v1.0 | None      | None         |
|           |                               |           |              |
| 456       | Cherry-picked to release/v2.0 | Conflict  | Dependency A |

**Data:**

```json
{
  "prs": [
    {
      "number": 123,
      "title": "Fix bug in module A",
      "labels": ["backport/stable", "bugfix"],
      "commits": [
        {"sha": "commit_sha_1", "message": "Fix bug #456"},
        {"sha": "commit_sha_2", "message": "Add logging to module B"}
      ]
    }
  ],
  "release_branches": ["release/v1.0", "release/v2.0"]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. PR Merge
6. Has 'backport/stable' Label?
7. Extract Rules
8. Skip PR
9. Cherry-Pick to ReleaseBranches
10. Resolve Conflicts
11. Check Dependencies?
12. Apply Prerequisites
13. Open Backport PRs

