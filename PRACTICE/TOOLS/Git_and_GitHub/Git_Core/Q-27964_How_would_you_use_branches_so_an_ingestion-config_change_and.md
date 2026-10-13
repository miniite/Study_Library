---
type: "practice"
difficulty: "Intermediate"
---

# How would you use branches so an ingestion-config change and a transformation-logic change can be reverted independently later?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Fetch vs Pull](<../../../../TOOLS/Git_and_GitHub/Git_Core/Fetch_vs_Pull.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to manage changes in ingestion configuration and transformation logic separately using Git branches. By creating specific branches for each type of change, we can revert them independently if needed. Here's how you would approach it:

TL;DR: Create a config-branch for handling ingestion-config changes and a transformation-branch for managing transformation-logic updates. These branches allow us to isolate the changes and revert them separately.

#### Triaging the Question

The question is essentially asking how to manage different types of changes in a Git repository such that they can be reverted independently.
We need to use Git branches effectively to separate these concerns, ensuring that any changes are tracked and can be easily rolled back if necessary.

#### First-Principles Solution

Start with the main master or main branch which contains the current stable codebase.
Create a new branch named config-branch for making ingestion-config changes.
SH
git checkout -b config-branch master
Make your necessary changes to the configuration files and commit them:
SH
# Modify ingestion_config.json or any relevant file
# Add, Commit, Push
git add .
git commit -m "Updated ingestion configuration"
git push origin config-branch
Similarly, create a branch named transformation-branch for transformation logic changes:
SH
git checkout -b transformation-branch master
Commit your transformation logic changes and push them to the repository:
SH
# Modify any relevant transformation code files
# Add, Commit, Push
git add .
git commit -m "Updated transformation logic"
git push origin transformation-branch

#### Closing Summary
+ Common Mistakes/Follow-ups:

By using separate branches (config-branch and transformation-branch), we can easily revert to a previous state of each component independently.
Ensure that both branches are well-documented with clear commit messages, making it easier for others (or your future self) to understand the changes.

#### Common mistakes
include:

Merging multiple types of changes into a single branch without proper documentation.
Not using descriptive and clear commit messages which can lead to confusion when reverting or reviewing changes.

#### Follow-ups
might involve discussing best practices like feature branches, pull requests, and code reviews for maintaining the quality and integrity of your codebase.

