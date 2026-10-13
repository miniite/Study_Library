---
type: "practice"
difficulty: "Deep"
---

# You track large model checkpoints with Git LFS, but a rollback to last week's tag pulls a checkpoint that LFS already garbage-collected from the remote. How would you couple LFS artifact retention to Git-referenced tags so any tagged model is always restorable?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Git LFS](<../../../../TOOLS/Git_and_GitHub/Git_Core/Git_LFS.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that any tagged model using Git Large File Storage (LFS) remains restorable, we need a robust strategy that couples LFS artifact retention with Git-referenced tags. The core idea is to configure Git LFS to retain artifacts based on the tags they are associated with, ensuring no tagged checkpoint gets garbage-collected until it's no longer referenced by any tag.

First, let's understand what Git LFS and tags do:

Git LFS tracks large binary files outside of your Git history.
Tags in Git are references to specific commits that remain immutable once created. They point to the commit hash at which a model checkpoint was saved.

Given the issue where rolling back to an old tag pulls a checkpoint that has been garbage-collected, we need to implement artifact retention rules that respect the tag lifecycle. This involves setting up LFS to retain artifacts as long as there's at least one tag referencing them.

#### Triage

The interviewer is probing:

Understanding of Git and Git LFS basics.
Knowledge of how tags work in Git.
Ability to implement a retention policy that ensures tagged checkpoints remain accessible.

#### Assumptions

The system uses Git for version control with large model checkpoints stored via Git LFS.
Tags are used to reference specific commit hashes (checkpoints) and are immutable once created.
We want to ensure that no tagged checkpoint can be garbage-collected as long as at least one tag references it.
Step-by-Step Solution

Configure Git LFS Retention Rules:

Use git lfs track command or configuration file to set up retention rules.
For each tag, associate the model checkpoint commit hash with a unique identifier (e.g., an alias in .lfsconfig).

Implement Tagging and Retention Logic:

When tagging a new commit:
BASH
git tag <tag_name> <commit_hash>
Ensure this command is part of the model training pipeline.
Update LFS retention rules to retain artifacts linked by tags:
BASH
# Example .lfsconfig entry
[lfs "track"]
  path = models/*
  alias = {ref="refs/tags/<tag_name>"}

Automate Retention Policy Enforcement:

Use a Git pre-push hook or a GitHub Action to enforce retention rules:
BASH
# Pre-push script
if git lfs track --verify <commit_hash> 2>/dev/null; then
  echo "LFS artifact retained for tag $tag_name"
else
  echo "Error: LFS artifact not retained for tag $tag_name, commit might be garbage-collected."
  exit 1
fi

Monitor and Maintain Retention Rules:

Regularly review tags to ensure they are still relevant.
Update .lfsconfig as needed when removing or adding tags.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By implementing a retention policy that couples LFS artifact retention with Git tags, we can ensure any tagged model checkpoint is always restorable. Common pitfalls include not properly updating the lfsconfig, accidentally garbage-collecting tagged checkpoints, and failing to automate enforcement.

#### Follow-up
questions might involve:

How would you handle multiple developers working on different versions of models?
What are potential performance impacts of this setup?
How do you integrate this with continuous integration (CI) pipelines?

