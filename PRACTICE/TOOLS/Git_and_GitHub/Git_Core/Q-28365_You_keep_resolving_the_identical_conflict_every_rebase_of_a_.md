---
type: "practice"
difficulty: "Deep"
---

# You keep resolving the identical conflict every rebase of a long-lived integration branch onto main. Explain how rerere records and replays resolutions, how to enable it repo-wide in CI, and its failure modes on subtly different conflicts.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Rerere and Repeated Conflicts](<../../../../TOOLS/Git_and_GitHub/Git_Core/Rerere_and_Repeated_Conflicts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, you're dealing with a common issue when integrating changes from multiple branches into your main branch using Git and GitHub. The rebase operation is repeatedly encountering the same conflict, which could be due to minor differences that aren't caught by the initial merge. To address this, we can leverage rerere (reuse recorded resolutions) records to automatically resolve these conflicts during future rebases.

#### Triage
What’s Being Probed

The interviewer is likely probing your understanding of:

How rerere works: Recording and replaying conflict resolution.
Enabling rerere in a CI/CD pipeline.
Identifying and mitigating failure modes, especially when subtle differences exist.

#### Assumptions
You are using Git for version control.
The conflict is recurring during rebases of an integration branch onto main.
Your team uses GitHub or a similar code hosting service.

#### First Principles Solution
What Is rerere?

rerere is a feature in Git that records and then replays your resolutions to conflicts, making future rebases smoother. When you resolve a conflict, these resolutions are stored as patterns, which can be applied automatically during subsequent rebases if similar conflicts arise.

Enabling rerere Repo-Wide

To enable rerere repo-wide:

Global Configuration (optional for one-time use):

BASH
git config --global rerere.enabled true

Local Repository Configuration:

To enable rerere on a per-repository basis, simply set the configuration in your local repository's .git/config file or using the command:
BASH
git config rerere.enabled true

Using CI/CD to Enable Repo-Wide in a Pipeline:

In a CI pipeline, you can configure it as part of the setup steps. For example, if using GitHub Actions, add this step to your workflow file:
YAML
- name: Enable rerere
  run: git config --local rerere.enabled true
How rerere Works

Recording Resolutions: When you resolve a conflict, Git records the resolution steps. These can be stored in the .git/review-cache directory.

Replaying Resolutions: During future rebases where similar conflicts arise, Git looks for recorded resolutions and applies them automatically.

#### Example
Workflow

#### Cons
ider you are resolving a merge conflict between two branches feature-X and main, and these steps are recorded. When the same conflict arises during another rebase from feature-X to main, Git will attempt to replay those recorded resolutions.

Failure Modes and Mitigations

Subtle Differences:

If the conflicts you are resolving in the future have subtle differences (e.g., slightly different filenames or paths), rerere might not be able to automatically resolve them.
Mitigation: Manually review recorded resolutions before applying, especially when subtle changes occur.

Overlapping Patterns:

If patterns from one conflict overlap with another in complex ways, it can lead to incorrect reapplication of previous resolutions.
Mitigation: Ensure that your conflict resolution process is well-documented and reviewed regularly.

Stale Records:

Over time, recorded resolutions might become stale due to changes in the project or the way conflicts are resolved.
Mitigation: Regularly clean up .git/review-cache and update patterns as needed.

#### Closing Summary

Enabling rerere can significantly improve your workflow by automating conflict resolution during rebases. However, it's crucial to be aware of its limitations, especially with subtle conflicts or changes over time.

#### Common Mistakes/Follow-Ups
Not enabling rerere: This might lead to manual resolution in every rebase.
Ignoring failure modes: Failing to review and update recorded resolutions can cause incorrect reapplication of patterns.
Over-reliance on automation: Always ensure you understand the conflicts and their context.

**Code:**

```bash
# Enabling rerere globally
git config --global rerere.enabled true

# Enabling rerere in a local repository
git config rerere.enabled true
```

**Flow:**

1. Yes
2. No
3. Enable Rerere
4. Is Rerere Enabled?
5. Record Conflict Resolutions
6. Manually Resolve Conflicts
7. Apply Recorded ResolutionsDuring Rebase
8. Review and Update Patterns asNeeded

