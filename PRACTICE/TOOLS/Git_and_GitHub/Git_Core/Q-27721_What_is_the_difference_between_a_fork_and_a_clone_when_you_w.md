---
type: "practice"
difficulty: "Basic"
---

# What is the difference between a fork and a clone when you want to contribute a fix to an open-source SQL toolkit?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Git vs GitHub](<../../../../TOOLS/Git_and_GitHub/Git_Core/Git_vs_GitHub.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In the context of contributing a fix to an open-source SQL toolkit, understanding the difference between a "fork" and a "clone" is crucial. Both actions are essential in Git and GitHub workflows, but they serve different purposes.

TL;DR:
Clone: You're making a local copy of the repository on your machine.
Fork: You’re creating a separate remote copy of the original repository under your account.

#### Triaging the Question

When you want to contribute a fix to an open-source SQL toolkit, you typically start by cloning the project's main repository onto your local machine. This allows you to work with the code on your own system and make changes as needed. However, if you need to propose these changes back to the original maintainers of the toolkit, you would create a fork of their repository. Forking creates a duplicate of the repository in your GitHub account, which is then used for submitting pull requests.

#### Assumptions
You have access to Git and GitHub.
The SQL toolkit has an open-source license allowing contributions.
Step-by-Step Solution from First Principles

Clone: Initially, you clone the main repository onto your local machine using a command like:

SH
git clone https://github.com/original-owner/sql-toolkit.git
cd sql-toolkit

This creates an exact copy of the project on your computer.

Make Changes: You work locally and make necessary changes to fix bugs or add features.

Commit & Push: Once you’ve made your changes, commit them:

SH
git add .
git commit -m "Fix for a specific SQL query issue"

Then push the commits back up to your local repository:

SH
git push origin main

Fork: To propose these changes to the original project, you first need to fork the repository on GitHub by clicking the “Fork” button. This action creates a copy of the repository under your account.

Clone Forked Repository: After forking, clone this new repository into your local environment:

SH
git clone https://github.com/your-username/sql-toolkit.git
cd sql-toolkit

Branching & Pull Requests: Create a new branch from the main or master branch in your forked repository for your changes:

SH
git checkout -b fix-sql-issue main

Make necessary commits and push them to your fork.

SH
git push origin fix-sql-issue

Create Pull Request: Finally, you go back to GitHub, navigate to the pull request tab in your forked repository, and create a new pull request against the original owner's main branch.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Common Mistake 1: Cloning directly from the original source without forking if you intend to contribute. This can lead to losing attribution or creating a messy history.
Common Mistake 2: Forgetting to create a new branch in your fork before making changes, which can result in a more complex merge process later.

In summary, cloning is used to work with the code on your local system, while forking allows you to contribute back to the original repository. Both processes are fundamental for engaging effectively with open-source projects.

