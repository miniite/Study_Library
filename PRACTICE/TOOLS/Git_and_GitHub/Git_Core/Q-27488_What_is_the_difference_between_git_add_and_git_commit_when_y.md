---
type: "practice"
difficulty: "Basic"
---

# What is the difference between git add and git commit when you're saving an edit to a Prometheus alert rule file?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Commit Messages](<../../../../TOOLS/Git_and_GitHub/Git_Core/Commit_Messages.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the differences between git add and git commit when working with a Prometheus alert rule file, we first need to grasp their roles in Git's workflow. In simple terms, git add stages changes, preparing them for the next commit, while git commit saves your staged changes as a new snapshot in Git’s history.

#### Triage

The question is essentially asking about the distinction between these two commands when editing a Prometheus alert rule file. The interviewer might be probing your understanding of version control and how it applies to managing configuration files like those used by Prometheus, which are crucial for monitoring systems.

#### Assumptions
You have made changes to a prometheus_rule.yml or similar file.
You want to save these changes as part of a commit in Git/GitHub.

#### First-Principles Solution

Let's break down the process step by step:

Edit the File: Suppose you make an edit to the alert rule configuration file, e.g., prometheus_rule.yml.

Stage Changes with git add:

git add is used to stage changes in your working directory for the next commit.
In practice, this means running git add <file> (e.g., git add prometheus_rule.yml) to tell Git which changes are ready to be committed.

Commit Changes with git commit:

git commit saves the staged changes as a new snapshot in the repository’s history.
This command requires you to provide a commit message explaining what was changed and why, e.g., git commit -m "Update alert rule for increased threshold".

#### Example
Workflow

Suppose we have made several minor adjustments to our Prometheus alert rules:

BASH
# Stage all changes in the file
$ git add prometheus_rule.yml

# Commit the changes with a descriptive message
$ git commit -m "Tweak thresholds and labels on critical alerts"

This sequence ensures that your changes are tracked properly, providing a clear record of what was done.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Mistake: Not using git add before committing can lead to accidental loss of changes or missing important details in the commit history.

#### Follow-Up
You might be asked about other Git commands relevant to workflow, such as git diff, git status, and how branching works with these commands.

By understanding these steps, you ensure that your version control practices are robust and maintain a clear history of changes made to critical monitoring configurations like Prometheus alert rules.

**Code:**

```bash
# Example Git commands
$ git add prometheus_rule.yml
$ git commit -m "Tweak thresholds and labels on critical alerts"
```

