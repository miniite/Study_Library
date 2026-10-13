---
type: "practice"
difficulty: "Deep"
---

# A revert of a bad migration merge to main is blocking the later re-merge of the corrected feature because Git thinks the changes are already applied. Explain the DAG mechanism and the exact sequence to cleanly re-introduce the migration.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Reverting Merges](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reverting_Merges.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this issue, we need to understand how Git's Directed Acyclic Graph (DAG) model works and then apply a specific sequence of commands to reintroduce the missing migration into the main branch without causing conflicts.

TL;DR

The problem arises because the bad migration merge to main has already been committed and pushed. To resolve this, you need to create a new commit that undoes these changes, ensuring the correct migration is then merged back into main.

#### Triaging the Question

What the Question Is Asking:

The question revolves around understanding Git's DAG structure.
It seeks knowledge on how to handle conflicts arising from merge commits in Git.
You need to explain the sequence of commands required to reintroduce a missing migration.

#### Assumptions

You are familiar with Git and have worked with branches, merges, and conflict resolution.
The bad migration is already merged into main.
The feature branch needs to be re-merged after correcting this issue.
Step-by-Step Solution

Identify the Commit Hashes:

Find the commit hash of the merge that introduced the bad migration.
Note down the commit hashes for both the feature and main branches before the problematic merge.

Create a New Branch to Revert:

SH
git checkout -b temp-branch <feature_branch>

Identify the Range of Commits to Revert:

Determine which commits are affected by the bad migration.
Use git log or gitk --all to visually inspect the commit history.

Create a Patch File:

SH
git format-patch <first_bad_commit_hash>^..<last_good_commit_hash>

This command generates a series of patch files representing the changes you need to revert.

Apply Patches to Temp Branch:

Navigate back to your feature branch or temporary working branch.
SH
git checkout temp-branch
Apply each patch file:
SH
git am <patch_file_1> <patch_file_2> ...

Force Push the Corrected Feature Branch:

Since you are changing history, force push to update the remote repository.
SH
git push origin temp-branch --force

Note that --force can overwrite the remote branch, so be cautious.

Re-Merge into Main:

Now merge your corrected feature branch back into main.
SH
git checkout main
git pull origin main  # Ensure you have the latest changes in main.
git merge --ff-only temp-branch

Resolve Any Merge Conflicts: If there are conflicts, resolve them manually and commit the resolution.

Push Changes to Remote Repository:

SH
git push origin main

#### Closing Summary

By following these steps, you can reintroduce the missing migration into main without disrupting the DAG structure of your Git repository. The key is to use Git's powerful tools like format-patch, am, and forced pushes to manage complex branch histories effectively.

#### Common Mistakes/Follow-Ups
Mistake: Failing to Verify Changes: Always double-check that all intended changes are reverted before force pushing.

#### Follow-Up
Handling Complex Merges: In more complex scenarios, consider using tools like git revert or git bisect for advanced conflict resolution.

**Code:**

```
SH
Copy
# Example commands to illustrate the process

git checkout -b temp-branch feature-branch  # Create a temporary branch from your feature branch
git format-patch <first_bad_commit_hash>^..<last_good_commit_hash>  # Generate patches for reverting bad changes
cd ..  # Navigate back to the original directory
git checkout temp-branch  # Switch to the temporary branch where you applied patches
git am <patch_file_1> <patch_file_2> ...  # Apply each patch file

# Force push the corrected feature branch
git push origin temp-branch --force

# Merge into main and resolve conflicts if any
git checkout main
git pull origin main  # Ensure latest changes in main are pulled
git merge --ff-only temp-branch  # Try to fast-forward merge, resolving conflicts as needed
git push origin main
```

**Flow:**

1. Start
2. Identify Bad Commit
3. Create New Branch to Revert
4. Identify Range of Commits
5. Create Patch File
6. Navigate Back and ApplyPatches
7. Force Push Corrected FeatureBranch
8. Merge into Main
9. Resolve Merge Conflicts if Any
10. Push Changes to RemoteRepository
11. End

