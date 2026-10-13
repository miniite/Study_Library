---
type: "practice"
difficulty: "Intermediate"
---

# How would you recover an earlier version of your alertmanager config after you overwrote it and committed a worse version three commits ago?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Recovering Lost Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Recovering_Lost_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To recover an earlier version of your AlertManager configuration in Git, especially when you've overwrote it with a worse version three commits back, we need to use Git's history traversal and reset capabilities. This involves understanding the commit history, using git checkout, or potentially rewriting the repository if needed.

The key steps are:

Identify the Commit Hash: Find the exact commit hash of the previous version of your AlertManager configuration.
Reset the Working Directory to That Version (or use an alternative method such as creating a new branch).

Now, let's go through these steps in detail:

#### Triaging the Question

The question is asking you to recover a specific file (alertmanager.yaml) from three commits back in your Git history. The interviewer may be probing your understanding of:

How Git's commit structure works.
How to use Git commands effectively for version control recovery.
Best practices for managing and recovering configurations.

#### Assumptions

Assume the following:

You are working on a branch named main.
Your repository contains an alertmanager.yaml file, which is crucial for your monitoring system.

#### First Principles Solution
Step 1: Identify the Commit Hash

First, we need to find the commit hash of the desired version of alertmanager.yaml. You can do this by:

Running git log --oneline -- alertmanager.yaml in the terminal.
Identifying the commit that contains the correct configuration.

Let's say the commit hash is abc123.

Step 2: Reset to That Version

Once you have the commit hash, you can use Git to reset your working directory to this version. There are a few ways to do this:

Using git checkout:

SH
git checkout abc123 -- alertmanager.yaml

This command will update the alertmanager.yaml file in your working directory with the contents of commit abc123.

Using a Temporary Branch: You might prefer to create a new branch from this commit for safety.

SH
git checkout -b temp-branch abc123

Now, you can make changes as needed and merge back into your main branch.

Step 3: Verify Changes

After making the necessary updates:

SH
git status

This will show you any untracked or modified files. You can then commit these changes if needed.

SH
git add .
git commit -m "Reverted to previous alertmanager configuration"

#### Closing Summary
and Common Mistakes/Follow-Ups

To summarize, recovering a specific version of your AlertManager config involves identifying the right commit hash using git log, and then either updating the working directory or creating a new branch from that commit. Always be cautious when overwriting files in production systems to avoid potential downtime.

#### Common Mistakes

Not backing up changes before making significant commits.
Overwriting important configuration files without version control.
Not understanding the implications of git checkout vs git reset.

#### Follow-Ups

How would you handle this if multiple team members are working on the same file?
What measures could be taken to prevent such issues in the future?

**Code:**

```
SH
Copy
# Example commands
git log --oneline -- alertmanager.yaml
git checkout abc123 -- alertmanager.yaml
git checkout -b temp-branch abc123
```

