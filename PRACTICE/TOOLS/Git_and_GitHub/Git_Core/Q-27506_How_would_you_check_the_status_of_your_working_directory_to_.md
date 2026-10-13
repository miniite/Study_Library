---
type: "practice"
difficulty: "Basic"
---

# How would you check the status of your working directory to see which dashboard and alert files you've changed before committing?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Reviewing Changes with Diff](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reviewing_Changes_with_Diff.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that you have a clear understanding of what changes you’ve made in your working directory, especially when dealing with dashboard and alert configurations in AIOps projects, checking the status is crucial before committing those files to Git or GitHub. This step helps avoid accidentally losing important work or pushing incomplete code. Here’s how you can do it:

Firstly, you need to be aware that Git is a version control system used to manage changes in software development and other processes. When working with configuration files for dashboards and alerts, you want to make sure your local changes are accurately tracked before committing them.

To check the status of your working directory, you can use the git status command in the terminal or Git Bash. This command will give you a clear indication of which files have been modified but not yet committed.

**Code:**

```bash
# Navigate to your project directory where .git folder exists
cd /path/to/your/aio-project

# Check the status of your working directory
git status
```

**Example input / output:**

```

Here’s a typical output you might see from running git status:

PLAINTEXT
Copy
On branch main
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)

	modified:   alerts/dashboard1.json
	modified:   dashboards/dash2.yaml

no changes added to commit (use "git add" and/or "git commit -a")

This output indicates that the files alerts/dashboard1.json and dashboards/dash2.yaml have been modified in your working directory but haven't yet been staged for a commit.
```

