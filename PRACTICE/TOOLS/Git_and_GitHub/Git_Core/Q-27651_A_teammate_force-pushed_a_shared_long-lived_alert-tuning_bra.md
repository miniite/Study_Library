---
type: "practice"
difficulty: "Deep"
---

# A teammate force-pushed a shared long-lived alert-tuning branch and three engineers' suppression-rule commits vanished from the tip right before an on-call rotation. As on-call owner, recover the lost commits with reflog and design guardrails so it can't recur.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Recovering Lost Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Recovering_Lost_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Force-pushes in Git/GitHub are a common source of conflicts when working on shared branches, especially in AIOps environments where rapid development cycles and multiple engineers collaborate closely. The scenario you described involves recovering lost commits by leveraging the reflog feature, which stores information about recent changes to the repository's refs (including branches). Additionally, we need to design guardrails to prevent such incidents from occurring again.

To recover the lost commits:

Understanding the Problem: The on-call engineers are concerned because three engineers' suppression-rule commits have disappeared right before an important rotation. The branch was force-pushed by a teammate, which means the commit history has been rewritten.
Using reflog to Recover Commits:
Git's reflog keeps track of all changes made to branches in your local repository.
We can use this information to find and recover the lost commits.

Here’s how you would proceed:

Check Local Reflog: First, check the local repository where the branch is checked out using git reflog to see if any recent operations have recorded the missing commits.

Identify Missing Commits:

SH
git reflog

This command will show a list of all actions that have been performed on this branch, including the commit hash and the state of the repository at each point in time.

Checkout and Restore: Once you identify the commit hashes from the reflog, checkout to an intermediary branch and use git reset --hard <commit_hash> to restore those commits.

Merge or Rebase: After restoring, merge them into your main alert-tuning branch if needed, or rebase onto the tip of the branch.

SH
# Example command sequence to recover missing commits
git checkout -b temp-branch  # Create a temporary branch
git reset --hard <commit_hash>  # Replace with actual commit hash from reflog
# Continue this for each lost commit
git checkout main_alert_tuning_branch
git merge temp-branch

To prevent such incidents in the future:

Branch Policy: Enforce strict policies on shared branches, such as not force-pushing unless absolutely necessary.
Automated Backups: Implement automated backups of critical branches to a different repository or branch.
Code Reviews and Check-ins: Encourage code reviews before merging into long-lived branches and use pull requests for any major changes.

To summarize: Reflog is a powerful tool for recovering lost commits, but it should be used as part of an overall strategy that includes strict branching policies, automated backups, and rigorous review processes to prevent such incidents from recurring.

**Code:**

```
SH
Copy
git reflog  # Check the local repository's reflog
git checkout -b temp-branch  # Create a temporary branch for recovery
git reset --hard <commit_hash>  # Replace with actual commit hash from reflog
```

**Example input / output:**

```text
$ git reflog
98765434 (HEAD -> main_alert_tuning_branch) HEAD~1:refs/heads/main_alert_tuning_branch: checkout: moving from temp to main_alert_tuning_branch
12345678 (temp-branch) 12345678: refs/heads/temp-branch: commit message details
```

**Flow:**

1. Identify lost commits using `gitreflog`
2. Checkout to a temp branch
3. Use 'git reset --hard<commit_hash>' to restore
4. Merge or rebase into mainbranch
5. Enforce strict policies andautomated backups
6. Implement code reviewprocess

