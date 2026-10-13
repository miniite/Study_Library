---
type: "practice"
difficulty: "Intermediate"
---

# How would you use git cherry-pick to apply a single urgent fix to a source-parsing bug onto your release branch without bringing over other in-progress changes?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Cherry-picking and Backports](<../../../../TOOLS/Git_and_GitHub/Git_Core/Cherry-picking_and_Backports.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the task of applying an urgent fix from a source-parsing issue directly into a release branch using git cherry-pick while avoiding unwanted in-progress changes, we need to understand and execute several key steps with precision. This process involves branching, committing, identifying the relevant commit, and then cherry-picking it selectively.

#### Triage

The question is essentially asking how to isolate and apply a specific patch or fix from a development branch (or any other non-release branch) directly into a release branch in Git, ensuring that only the necessary changes are merged without bringing over additional, unfinished work. The interviewer might be probing your understanding of Git workflows, branching strategies, and cherry-picking mechanics.

Step-by-Step Solution
Identify the Commit: First, locate the specific commit that contains the fix for the source-parsing bug on the development branch.
Create a New Branch (if not already): To avoid modifying the release branch directly, create a new temporary branch from the release branch.
Cherry-Pick the Commit: Apply the identified commit to this new branch using git cherry-pick.
Resolve Conflicts: If there are any conflicts between the fix and existing code on the release branch, resolve them manually or through an interactive rebase.
Test the Fix: Thoroughly test the change in a staging environment before merging it back into the release branch.
Merge to Release Branch: Once tested and verified, merge this change into the release branch.
Detailed Workflow
Locate the Commit ID:
SH
git log --grep="source-parsing bug fix"
Create a New Branch from the Release Branch (if not already):
SH
git checkout -b temp-fix-branch release-branch
Cherry-Pick the Specific Commit:
SH
git cherry-pick <commit-id>
Resolve Conflicts: If conflicts arise, use git mergetool to resolve them.
Test in Staging Environment: Perform thorough testing to ensure the fix works as expected.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By carefully selecting and cherry-picking only the necessary commit, you can efficiently apply urgent fixes directly into a release branch without disturbing other ongoing work. Common mistakes include inadvertently merging unrelated changes or not thoroughly testing the applied fix before deployment. Follow-ups could involve discussing how to handle more complex merge scenarios or alternative methods like using Git rebase for better integration of patches.

**Code:**

```
SH
Copy
# Step 1: Identify the commit ID
git log --grep="source-parsing bug fix"

# Step 2: Create a new branch from release-branch
git checkout -b temp-fix-branch release-branch

# Step 3: Cherry-pick the specific commit
git cherry-pick <commit-id>

# Step 4: Resolve any conflicts if needed
git mergetool

# Step 5: Test in staging environment (assuming a script to run tests)
./test-suite.sh
```

**Example input / output:**

```text
# Sample command sequence
$ git log --grep="source-parsing bug fix"
commit abc1234567890abcdef1234567890abcde (HEAD -> release-branch, origin/release-branch)
Author: Jane Doe <jane.doe@example.com>
Date:   Mon Jan 1 10:00:00 2024 -0800

    Fix parsing bug in src/parser.py

# Create and switch to a new temporary branch
$ git checkout -b temp-fix-branch release-branch

# Cherry-pick the commit into the new branch
$ git cherry-pick abc1234567890abcdef1234567890abcde
```

