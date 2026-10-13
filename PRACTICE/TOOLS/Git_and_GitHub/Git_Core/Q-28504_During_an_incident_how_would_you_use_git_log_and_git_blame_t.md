---
type: "practice"
difficulty: "Intermediate"
---

# During an incident, how would you use git log and git blame to find which change likely introduced a config regression?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Blame and Ignore Revisions](<../../../../TOOLS/Git_and_GitHub/Git_Core/Blame_and_Ignore_Revisions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, I'll start with a quick overview of what we're looking to achieve. In an emergency situation where a configuration change has caused a regression in our application, git log and git blame are crucial tools that can help us pinpoint the problematic commit and line of code. Let's dive into how these commands work and then solve the problem step by step.

TL;DR
Use git log to identify commits affecting specific files.
Utilize git blame on those files to find which exact lines were modified in a particular commit.

#### Triaging the Question

The question is essentially asking us to use Git commands (log and blame) to diagnose an issue where a recent configuration change has caused a regression. The interviewer is likely testing our understanding of these tools and their practical application in debugging code.

#### Assumptions
We have access to the repository and can run Git commands.
The regression was introduced after a known commit or merge point.
We are dealing with changes made to configuration files, not necessarily application code.

#### First Principles Solution
Identify Affected Files: Start by using git log to find commits that touch the relevant configuration file(s).
Analyze Changes Line by Line: Once we have identified suspicious commits, use git blame on those files to see which lines were modified in each commit.
Step 1: Identifying Affected Commits
BASH
git log -- path/to/config/file.conf

This command will show us a list of commits that touched the configuration file. Look for recent changes that could have caused the regression.

Step 2: Analyzing Changes with git blame

Once we've identified a suspicious commit, let's take a closer look at its changes using git blame:

BASH
git blame -L 'line-number1,line-number2' path/to/config/file.conf

Here, replace line-number1 and line-number2 with the actual line numbers you are interested in. If we don't know the exact lines, we can just use:

BASH
git blame --show-patch path/to/config/file.conf

This will output a patch of changes for each commit that touched the file, showing which lines were modified by whom and when.

#### Example
Walkthrough

Let's assume the following steps:

Identify Suspect Commit:

BASH
git log -- path/to/config/application.yml

Suppose we find a commit abc123 from 15 minutes ago that changed application.yml.

Analyze Changes:

BASH
git blame -L '20-30' path/to/config/application.yml

This command will show us the changes made between lines 20 and 30 in that commit.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By using git log and git blame, we can efficiently narrow down the source of a configuration regression. Common mistakes include overlooking the context of the change (e.g., not checking if other files were affected) or assuming the most recent changes are always responsible.

Triaging Follow-Up Questions
What if there's no clear commit causing the issue?
How would you handle large repositories with many contributors?
Are there any tools that can automate this process for larger codebases?

**Code:**

```bash
git log -- path/to/config/file.conf
git blame -L 'line-number1,line-number2' path/to/config/file.conf
```

