---
type: "practice"
difficulty: "Deep"
---

# Two release branches of the serving stack both need the same model-safety patch but the surrounding code diverged. Compare cherry-pick -x, rebase --onto, and format-patch/am for landing it on each, and when each is preferable for provenance.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Cherry-picking and Backports](<../../../../TOOLS/Git_and_GitHub/Git_Core/Cherry-picking_and_Backports.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with integrating a model-safety patch across divergent branches in Git, understanding cherry-pick -x, rebase --onto, and format-patch/am is crucial. Each method has its own strengths and weaknesses, particularly in terms of provenance and handling divergent codebases.

#### Triage
Understanding the Problem

The core issue here involves integrating a change (model-safety patch) into two separate release branches where the surrounding code has diverged significantly. The goal is to do this efficiently while maintaining accurate history and clear provenance.

Cherry-Pick -x

#### Overview
cherry-pick allows you to apply a specific commit from one branch onto another. When combined with -x, it retains more of the original commit message, preserving historical context.

Steps:

Identify the model-safety patch commit.
Run git cherry-pick -x <commit-hash> on each release branch.

#### Pros

Preserves exact commit hash and original authorship details.
Easy to identify the patch among other commits.

#### Cons

Doesn't handle divergent code well, might require additional conflict resolution.
Less flexible if you need to adjust the patch later.
Rebase --onto

#### Overview
rebase --onto is used when the source and target branches have diverged significantly. It helps in integrating changes from a specific point without affecting history much.

Steps:

Create a temporary branch from the model-safety commit.
Run git rebase --onto <target-branch> <source-branch> <temporary-branch> on each release branch.

#### Pros

Can handle significant divergences in codebases effectively.
Results in cleaner history as it moves commits rather than applying them directly.

#### Cons

More complex to use, requires careful handling of conflicts.
Can rewrite commit hashes if not done carefully.
Format-Patch/Am

#### Overview
format-patch creates a patch file that can be applied later using am. This method is flexible and allows for easy handling of divergent branches by applying the changes selectively.

Steps:

Generate the model-safety patch with git format-patch <commit-hash>.
Apply the generated patch on each release branch using git am.

#### Pros

Highly flexible and customizable.
Allows for selective application of patches, useful in complex merges.

#### Cons

More manual work compared to other methods.
Risk of misapplying or losing context during the process.
Choosing the Best Method

Cherry-Pick -x: Prefer this when you need a clean and simple merge with minimal history rewriting, and the divergences are minor. It's straightforward for small patches but might become cumbersome if dealing with larger changes.

Rebase --onto: Optimal for significant divergences where you want to maintain clean history. Useful in complex environments like large codebases or when multiple branches have evolved differently.

Format-Patch/Am: Ideal for complex scenarios requiring flexible patch handling and selective application. Provides detailed control over the integration process, which is invaluable in intricate projects but requires more effort and attention to detail.

#### Closing Summary

In summary, cherry-pick -x is best when you want a simple and direct merge with minimal history rewriting. rebase --onto excels for significant divergences while maintaining clean history. format-patch/am offers the highest flexibility but requires more effort in application.

#### Common Mistakes
Overlooking conflict resolution: Ensure thorough testing post-integration.
Misapplying patches: Double-check that the patch is correctly applied to avoid introducing errors.
Inconsistent provenance: Use methods that maintain clear history and context for better traceability.

**Comparison:**

```
Method	Pros	Cons
cherry-pick -x	Preserves exact commit hash and original authorship details.	Doesn't handle divergent code well, requires conflict resolution.
rebase --onto	Can handle significant divergences in codebases effectively.	More complex to use; can rewrite commit hashes if not handled carefully.
format-patch/am	Highly flexible and customizable for selective application.	More manual work compared to other methods, risk of misapplication.
```

**Code:**

```bash
# Example cherry-pick -x command
git cherry-pick -x <commit-hash>

# Example rebase --onto command
git checkout <target-branch>
git rebase --onto <target-branch> <source-branch> <temporary-branch>
```

