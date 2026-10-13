---
type: "practice"
difficulty: "Intermediate"
---

# Why are local Git hooks not shared with your team by default, and how could you distribute a SQL-formatting hook everyone on the data team uses?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Pre-commit Hooks](<../../../../TOOLS/Git_and_GitHub/Git_Core/Pre-commit_Hooks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Local Git hooks are specific to each developer's environment and are not typically shared because they perform actions based on individual setups or preferences. To ensure consistency and collaboration, we can create a common SQL formatting hook that is shared among the team using GitHub. This involves creating a custom hook in your repository, committing it, and then ensuring everyone clones or pulls from this central location to use the same hooks.

#### Triage

The question probes understanding of Git hooks, their purpose, and how they are used in a collaborative environment like a data team. The core idea is to ensure that all team members follow consistent coding standards without needing to individually configure each machine.

#### First-Principles Solution

Understanding Local Hooks:

Local Git hooks are files stored in the .git/hooks directory of your local repository.
These hooks run scripts or commands before and after common Git operations like commit, push, etc., to automate tasks such as formatting, linting, or logging.

Why Not Share Locally?

Local hooks are specific to individual setups because the environment (e.g., operating system, tools installed) can vary.
Sharing local hooks would require everyone to have identical configurations, which is impractical and can lead to issues if someone has a different setup.

Creating a Shared Hook:

Develop a SQL formatting hook using any language of your choice (e.g., Python, Bash).
Place this script in the repository's .github directory if you are using GitHub Actions or at a location that will be shared across all team members.

Implementing the Hook:

Create a new file in the .git/hooks/pre-commit directory within your repository.
Make sure to name it pre-commit.sample initially (this is a convention) and rename it to pre-commit.
In this file, call out your custom SQL formatting script.

Ensuring Everyone Uses It:

Use GitHub Actions or GitLab CI/CD pipelines to enforce the use of these hooks.
Alternatively, you can inform your team members to manually clone or pull from the repository and add a symbolic link to their local .git/hooks directory if needed.

#### Closing Summary

By centralizing the SQL formatting hook in the repository's .github directory, we ensure that all team members use the same standards without needing individual setups. This approach leverages GitHub’s collaboration features to maintain consistency across the development process.

#### Common Mistakes/Follow-Ups

Mistake 1: Forgetting to commit and push your custom hook.

#### Follow-Up
1: Ensure you have a backup plan if someone needs to work on an offline machine without internet access.

Mistake 2: Ignoring environment-specific dependencies in the hook.

#### Follow-Up
2: Use git clone --recursive or update hooks via GitHub Actions to handle any necessary dependencies.

Mistake 3: Not providing clear documentation for new team members.

#### Follow-Up
3: Document the process and include instructions on how to set up and use the hook in your project's README file.

