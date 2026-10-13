---
type: "practice"
difficulty: "Deep"
---

# A rebase of a long-lived infra feature branch onto main keeps re-conflicting on the same generated lockfile every commit. Diagnose why, and decide between rerere, a merge driver that regenerates the lockfile, or squashing before rebase.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Rerere and Repeated Conflicts](<../../../../TOOLS/Git_and_GitHub/Git_Core/Rerere_and_Repeated_Conflicts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we are dealing with a Git workflow issue where rebasing a long-lived feature branch onto main results in constant conflicts due to a generated lockfile. The goal is to diagnose the root cause and choose an appropriate solution among rerere, a custom merge driver that regenerates the lockfile, or squashing before rebase.

Triaging the Problem

The core issue here is understanding why Git repeatedly reports conflicts for a file that should be automatically generated. Let's break down the key aspects:

Generated Lockfiles: These are files that are created during the build process and are typically not version-controlled because they contain state information.
Rebase Process: Rebasing involves applying all commits from one branch onto another, which can lead to conflicts if there is a conflict in the code or generated content.
Diagnosing the Issue

The frequent conflicts suggest that the lockfile is being regenerated differently during each rebase attempt compared to when it was originally committed. Here are some steps to diagnose further:

Check Commit History: Inspect recent commits on both branches (main and the feature branch) for any changes in how the lockfile is generated.
Diff Output: Use git diff or a tool like meld to compare the contents of the lockfile across different commits to see if there are any subtle differences.

#### Solution
Approaches

Given the nature of the problem, we can evaluate the pros and cons of each solution:

Rerere (Re-Record-Eric):

Explanation: rerere is a Git feature that records the result of resolving conflicts during interactive rebase or merge. It can then automatically resolve similar conflicts in the future.

#### Pros
Simplifies repetitive conflict resolution by remembering previous resolutions, making it easier to complete rebase operations without manual intervention.

#### Cons
May not fully address the root cause if the lockfile is generated differently each time.

Custom Merge Driver:

Explanation: A custom merge driver can be written in a language like Bash or Python that regenerates the lockfile based on current conditions, ensuring consistency during rebase.

#### Pros
Ensures the lockfile is always up-to-date and consistent across branches, potentially preventing conflicts from occurring.

#### Cons
Requires additional development effort to create and maintain the custom driver.

Squashing Before Rebase:

Explanation: Squashing involves combining multiple commits into a single commit before rebasing. This can simplify the rebase process by reducing the number of conflicts.

#### Pros
Reduces the complexity of the rebase, making it less error-prone and easier to manage.

#### Cons
May lead to less granular history if not done carefully.
Decision Criteria
Frequency and Severity of Conflicts: If frequent conflicts are due to minor differences in lockfile generation, rerere might suffice. However, if the differences are significant or complex, a custom merge driver could be more appropriate.
Team Experience with Custom Solutions: If your team has experience with writing custom Git drivers, this approach can provide long-term benefits. Otherwise, squashing might offer simpler management in the short term.

#### Closing Summary
and Common Mistakes

In summary, the decision between rerere, a merge driver that regenerates the lockfile, or squashing before rebase depends on the frequency and nature of the conflicts, as well as team expertise. A thorough understanding of Git workflows and tools like rerere is crucial for making an informed choice.

#### Common mistakes
include:

Overlooking subtle differences in how the lockfile is generated.
Relying too heavily on rerere without addressing the root cause of conflicting files.
Failing to consider the long-term maintenance implications of custom merge drivers.

**Comparison:**

```
Solution	Pros	Cons
Rerere	Simplifies conflict resolution.	May not fully address root cause.
Custom Merge Driver	Ensures consistent lockfile generation.	Requires development effort.
Squashing Before Rebase	Reduces complexity of rebase process.	Can reduce granular history.
```

**Example input / output:**

```text
# Example of using `git rerere` to handle conflicts
$ git checkout feature-branch
$ git rebase main

# Conflicts are recorded by rerere
$ git config --global rerere.enabled true
$ git config --global rerere.autoupdate true
```

**Flow:**

1. Yes
2. No
3. Start
4. Is the conflict due to minordifferences?
5. Rerere (Re-Record-Eric)
6. Custom Merge Driver
7. Squashing Before Rebase
8. End with simplified rebase

