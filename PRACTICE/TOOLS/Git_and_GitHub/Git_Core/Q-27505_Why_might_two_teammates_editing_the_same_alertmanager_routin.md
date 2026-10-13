---
type: "practice"
difficulty: "Intermediate"
---

# Why might two teammates editing the same alertmanager routing file on different branches cause a merge conflict, and how would you resolve it?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Merge Conflicts](<../../../../TOOLS/Git_and_GitHub/Git_Core/Merge_Conflicts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When two teammates are working on the same alertmanager.yml file in different branches of a Git repository, they might encounter merge conflicts due to overlapping changes made by both developers. This situation arises because Git needs to resolve differing versions of the same code or configuration file during the merge process.

#### Triage

The question is asking why merge conflicts occur and how to handle them. The core of this issue revolves around understanding Git's merge mechanism, which identifies conflicting changes when branches diverge. We will assume that both developers are aware of Git basics but need guidance on handling specific types of conflicts.

#### First-Principles Solution
Identify the Conflict: When attempting to merge the two branches, Git will identify lines in alertmanager.yml where the content differs between the base (common ancestor) and each branch.
Resolve Conflicts Manually: Developers need to manually review and decide on the correct changes or a merged version of both conflicting lines.
Commit Resolved Changes: Once the conflict is resolved, commit the changes with an appropriate message.
Step-by-Step Solution
Check for Merge Conflicts:
BASH
git checkout feature-branch  # Switch to one branch where you want to merge into main or other branches.
git merge development-branch  # Attempt to merge from another branch (e.g., 'development-branch').
Identify Conflict in alertmanager.yml:
BASH
git status  # Shows the files with conflicts, e.g., alertmanager.yml.
Edit Conflicted File: Open the conflicted file and resolve the conflict by hand. For example:
YAML
#<<<<<
- job_name: 'prometheus'
  static_configs:
    - targets: ['localhost:9090']

#<<<<<<< development-branch
- job_name: 'kubernetes'
  kubernetes_sd_configs:
    - role: pod
#=======
- job_name: 'kubernetes'
  kubernetes_sd_configs:
    - role: node
#>>>>>>> feature-branch
Resolve with git add:
BASH
git add alertmanager.yml  # Add the resolved file to the staging area.
Commit Resolved Changes:
BASH
git commit -m "Resolved merge conflict in alertmanager routing"  # Commit the changes.

#### Common Mistakes
and Follow-Ups
Not Properly Staging Changes: Forgetting git add after resolving conflicts can lead to a partially merged repository.
Ignoring Conflicts: Ignoring Git’s warning about conflicting files can result in incomplete merges or even unmerged code.
Misunderstanding Merge Strategies: Using automated merge tools without understanding the underlying changes might not always produce the desired outcome.

#### Closing Summary

In summary, when two teammates are working on the same file in different branches of a Git repository and both make overlapping changes, conflicts can arise during merges. Properly identifying these conflicts, resolving them manually, and committing the resolved files ensures that the team's work is properly integrated without loss or corruption of code.

