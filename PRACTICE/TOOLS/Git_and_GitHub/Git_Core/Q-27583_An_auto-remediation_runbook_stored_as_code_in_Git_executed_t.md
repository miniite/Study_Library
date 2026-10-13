---
type: "practice"
difficulty: "Deep"
---

# An auto-remediation runbook stored as code in Git executed the wrong scale-down because a merge took the wrong side of a conflict in a threshold guard. How would you diff the merge commit against both parents to audit the resolution and reproduce the intended safe behavior?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Merge Conflicts](<../../../../TOOLS/Git_and_GitHub/Git_Core/Merge_Conflicts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this issue, we need to understand that Git’s merge process can sometimes lead to incorrect resolutions when conflicts arise. In your scenario, a merge conflict in the threshold guard logic led to an unintended scale-down operation during auto-remediation. The task at hand is to audit and reproduce the intended safe behavior by diffing the merge commit against both parents.

#### Triage

The question probes how well you can use Git commands to analyze merge conflicts and their resolutions, ensuring that the correct version of code is executed post-merge. Key assumptions include:

You are familiar with Git’s merge process.
The repository contains the auto-remediation runbook as code.
A specific merge commit introduced a conflict in the threshold guard logic.

#### Solution
Step 1: Identify the Merge Commit

First, we need to identify the merge commit that introduced the issue. Assuming the commit hash is merge-commit-hash, you can use Git commands to inspect this commit and its parents:

BASH
git log --oneline -p merge-commit-hash

This command will show a detailed diff of the merge commit.

Step 2: Inspect Both Parents

To understand how each parent handled the conflict, we need to check out both parents separately. Let’s assume the parent commits are parent1 and parent2. You can use the following commands:

BASH
git checkout parent1

After checking out parent1, you can compare it with the merge commit using:

BASH
git diff merge-commit-hash

Similarly, check out parent2 and compare it with the merge commit:

BASH
git checkout parent2
git diff merge-commit-hash
Step 3: Audit the Conflict Resolution

By comparing these diffs, you can audit how Git resolved the conflict. Typically, Git takes one side of the conflict (either from parent1 or parent2) and merges it into the final commit. The goal is to determine which parent’s logic was taken in this merge.

Step 4: Reproduce the Intended Behavior

To ensure that the intended safe behavior is maintained, you should consider a few strategies:

If the conflict resolution took one side of the threshold guard by mistake, you can manually apply the correct version.
You might need to re-run the auto-remediation process with the corrected code.
Document the issue and its fix for future reference.
Step 5: Test the Changes

Before deploying any changes, thoroughly test the modified runbook to ensure it behaves as intended. This could involve running simulations or performing a dry run in a non-production environment.

#### Closing Summary

By following these steps, you can effectively audit and reproduce the correct behavior of your auto-remediation runbook after a merge conflict. The key is understanding how Git resolves conflicts and comparing the parents with the merged commit to identify any discrepancies.

#### Common Mistakes/Follow-Ups
Misinterpreting Conflict Resolution: Always double-check which side of the conflict was taken in the merge.
Failing to Test Thoroughly: Ensure that all edge cases are covered before deploying the fix.
Ignoring Documentation: Document the issue and resolution for future reference.

**Code:**

```bash
git log --oneline -p merge-commit-hash
git checkout parent1
git diff merge-commit-hash
git checkout parent2
git diff merge-commit-hash
```

