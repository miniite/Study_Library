---
type: "practice"
difficulty: "Deep"
---

# Your team debates squash-merge versus merge-commit versus rebase-merge as the default for the pipeline repo. Argue the trade-offs for bisecting a data-quality regression, per-change revert granularity, and history readability, and make a defensible recommendation.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Merge Strategies](<../../../../TOOLS/Git_and_GitHub/Git_Core/Merge_Strategies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When considering squash-merge, merge-commit, and rebase-merge as default strategies for our pipeline repo, we need to weigh several factors including bisecting a data-quality regression, per-change revert granularity, and history readability. Let's break down each aspect from first principles.

TL;DR

Squash-merge consolidates multiple commits into one before merging, making the commit history cleaner but harder to trace individual changes. Merge-commit retains all changes in separate commits, providing detailed change history at the cost of a more fragmented history. Rebase-merge rewrites the commit history by applying current changes on top of the latest base branch, which can be useful for maintaining linear commit histories.

#### Triage

The primary question here is to evaluate and recommend a Git workflow strategy that balances bisecting capabilities, revert granularity, and overall history readability. Let's explore each in detail:

Bisecting a Data-Quality Regression:

Bisecting involves using git bisect to automatically narrow down the range of commits causing an issue.
Squash-merge results in fewer commits, making it harder for Git to bisect accurately because it loses context between changes.

Per-Change Revert Granularity:

Merge-commits allow easy reverting individual or small sets of changes by targeting specific commits.
Squash-merge and rebase-merge can obscure individual change boundaries, making reverts more cumbersome.

History Readability:

Merge-commits preserve a chronological history with clear commit messages for each change.
Squash-merge simplifies the history but at the cost of losing detailed information about specific changes.
Rebase-merge can offer a clean linear history but also risks altering historical context.

#### First-Principles Solution
Bisecting a Data-Quality Regression
Merge-commits: Provide the best bisecting capability because each commit has its own message and changes. This makes it easier to pinpoint issues by looking at the commit details.
Squash-merge & Rebase-merge: These strategies compress multiple changes into fewer commits, complicating bisecting efforts as context is lost.
Per-Change Revert Granularity
Merge-commits: Best for granular rollbacks and debugging since each change can be isolated easily.
Squash-merge & Rebase-merge: More complex to revert individual changes due to fewer, larger commits. This could lead to cascading effects if a misstep occurs.
History Readability
Merge-commits: Maintain the most readable history as each commit has its own message and context.
Squash-merge & Rebase-merge: Provide cleaner histories but sacrifice detailed change information.
Recommendation

Given our need for both robust bisecting capabilities and fine-grained revert granularity, merge-commits should be the default strategy. Here’s why:

Bisecting Capabilities:

git bisect works optimally with separate commits, allowing us to trace changes accurately.

Revert Granularity:

Easy to revert specific changes or a set of small changes using git revert <commit>.

History Readability:

Chronological and detailed commit history ensures that each change can be independently verified and understood.

#### Closing Summary

In conclusion, merge-commits are the best default for our pipeline repo because they support effective bisecting, enable fine-grained reverts, and maintain a clear and readable history. However, we should also consider hybrid strategies where necessary, such as using squash-merge for clean-up commits or rebase-merge for maintaining linear histories in certain sub-repos.

#### Common Mistakes/Follow-Ups
Overlooking Bisecting Capabilities: Some teams might favor cleaner commit histories over the ability to bisect accurately.
Ignoring Granular Reverts: Focusing solely on history readability without considering ease of reversion can lead to operational inefficiencies.
Ignoring Context Loss: Squash-merge and rebase-merge can obscure important context, making debugging and troubleshooting harder.

**Code:**

```python
# Example Git command for bisecting a data-quality issue
git bisect start
git bisect bad # Mark the current HEAD as the "bad" commit
git bisect good <commit_hash> # Mark an earlier known good commit
```

**Flow:**

1. Yes
2. No
3. Start Bisect
4. Is Merge-Commit Used?
5. Accurate Bisecting
6. Squash or Rebase-Merge
7. Granular Revert Capabilities
8. Cleaner History butCompromised Change Context

