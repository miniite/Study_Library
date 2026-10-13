---
type: "practice"
difficulty: "Intermediate"
---

# Why do submodules pin a specific commit rather than track a branch, and what pain does that cause during updates?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Fetch vs Pull](<../../../../TOOLS/Git_and_GitHub/Git_Core/Fetch_vs_Pull.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When using Git submodules, each submodule can be pointed to a specific commit, not a branch. This approach is designed for situations where you want to lock down dependencies at a precise version or when you need reproducibility in your projects. However, this choice comes with its own set of challenges, particularly during updates and maintenance.

Firstly, let's break down the core concept: submodules are essentially Git repositories inside another repository. When you initialize a submodule, you specify a commit hash that pinpoints an exact version of the dependency. This is different from tracking branches because branches evolve; commits move as developers push new changes or revert old ones.

Now, why do we pin to a specific commit? The main reasons are:

Stability and Reproducibility: By locking down dependencies to specific versions, you ensure that your project's state at the time of development can be exactly replicated in the future.
Dependency Management: Certain projects or libraries may not have stable branches yet; they might be under heavy development, and specifying a commit ensures you get the exact functionality you need.

However, pinning to a specific commit can cause pain during updates because:

Loss of Branch Tracking: If the main project needs an update to a newer version of a dependency, but that new version introduces breaking changes or has different functionalities, updating the submodule to its latest commit might not be straightforward.
Manual Effort: You need to carefully manage which commits are compatible and make manual adjustments when updates are necessary.

For example, if you have a project mainrepo with a submodule submoduleA, and submoduleA is pinned at commit abc123. If the latest version of submoduleA has significant changes or fixes that are crucial for your main project but introduces breaking changes, you need to find an equivalent commit or manually merge changes.

Here's how this works in practice:

You might initially clone a submodule with git submodule add <url>, specifying a particular commit.
Later, if the project evolves and a new branch or tag is more suitable, you can still update it by first checking out the desired branch/tag of the submodule repository and then updating the .gitmodules file in your main repo.

Let's consider a simple scenario:

BASH
# Initial setup with a specific commit
git clone https://github.com/user/mainrepo.git
cd mainrepo
git submodule add -b v1.0.0 https://github.com/user/submoduleA.git path/to/submodule

# Later, when trying to update the submodule to a newer version
cd path/to/submodule
git checkout master  # Assuming you want to track the latest master branch now
git pull origin master  # Fetch and merge new changes

In summary, while pinning submodules to specific commits provides stability and reproducibility, it can complicate updates due to dependency evolution and potential breaking changes.

**Code:**

```bash
# Example of adding a submodule with a specific commit
git submodule add -b v1.0.0 https://github.com/user/submoduleA.git path/to/submodule

# Updating the submodule from its current commit
cd path/to/submodule
git checkout master  # Switch to tracking the latest branch
git pull origin master  # Fetch and merge new changes
```

**Example input / output:**

```text
# Initial setup
Cloning mainrepo...
Cloning submoduleA at v1.0.0...

# Attempting update
Updating submodule to the latest branch...
Switched to branch 'master'
Your branch is up-to-date with 'origin/master'.

git pull origin master  # Output: Already up to date.
```

**Flow:**

1. Yes
2. No
3. Initial Setup
4. Update Required?
5. Manually update submodulecommit
6. Submodule is up to date
7. Review changes, testcompatibility
8. Apply necessary changes inmain project

