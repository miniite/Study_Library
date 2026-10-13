---
type: "practice"
difficulty: "Intermediate"
---

# A junior ran git reset --hard on the alert-correlation branch and thinks a day of tuning work is gone. Walk through recovery via reflog and fsck --lost-found, and the exact conditions under which those correlation-rule commits are truly unrecoverable.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Recovering Lost Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Recovering_Lost_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a junior runs git reset --hard on a branch, it can result in losing uncommitted changes or recent commit history. In this scenario, they might have lost work done over a day. Recovery involves using Git's built-in features like the reflog and fsck --lost-found to try to retrieve the missing commits. Let’s walk through these steps and understand when the recovery is possible.

#### Triage

The question asks us to recover from git reset --hard on an alert-correlation branch, where a day of tuning work has been lost. The key here is to use Git's reflog and fsck commands to find any lingering data that might have been moved or deleted.

#### Assumptions
The junior ran git reset --hard on the alert-correlation branch.
The commits containing the day of tuning work are no longer in the branch but could still be found in the reflog and lost-found repository.

#### Solution
Step 1: Reviewing the Reflog

The Git reflog is a record of all changes to the tip of branches and other references, including reset --hard. This can help us find the SHA of the commit just before the git reset --hard was performed.

SH
git reflog

This command will show you a list of recent operations on the branch. Look for an entry that matches the day's work and note its SHA.

Step 2: Reverting to the Last Good State

Once we have the commit hash from the reflog, we can revert to that state using git reset or git checkout.

SH
# Using git reset to set the HEAD pointer back
git reset --hard <commit_hash>

# Or directly checkout the commit if you want a clean working tree
git checkout <commit_hash>
Step 3: Recovering Lost Files with fsck --lost-found

If the reflog does not show any useful information, or if the commits have been lost completely from the branch history, we can use git fsck to check for any dangling objects in the repository.

SH
git fsck --full

This will give us a list of all dangling objects (commits, blobs, trees) that are not pointed to by any reference. We need to check if any of these objects correspond to the day's tuning work.

To recover from fsck --lost-found, we can use:

SH
git fsck --lost-found | grep lost-found

This will list all the objects found in the lost-found repository, which might contain our missing commits. We can then attempt to checkout these objects directly.

SH
git cat-file -p <object_hash> > recovered_file.txt
# This command shows the content of the object and saves it to a file for inspection or recovery.
Step 4: Understanding When Recovery is Unlikely

Git’s reflog and fsck are powerful tools, but there are conditions under which recovery might not be possible:

Commit has been deleted from the repository: If the commit was forcefully removed using git filter-branch or other destructive commands.
Repository prune policy: If the repo's remote origin has pruning enabled to delete old data after a certain period, those commits may have already been pruned.

#### Closing Summary

In summary, recovery from a git reset --hard can often be achieved using Git’s reflog and fsck --lost-found commands. However, there are scenarios where the recovery might not be possible due to destructive actions or pruning policies.

#### Common Mistakes
and Follow-Ups
Overlooking the use of reflog: Many forget that git reflog is a valuable tool for recovering lost commits.
Misunderstanding fsck --lost-found outputs: It’s easy to miss the relevant dangling objects in the output unless you understand what each line represents.
Not checking remote repositories: If the work was pushed to another repository, it might still be recoverable from there.

**Code:**

```
SH
Copy
git reflog
git fsck --full
git fsck --lost-found | grep lost-found
```

