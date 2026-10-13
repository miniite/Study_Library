---
type: "practice"
difficulty: "Basic"
---

# How would you configure Git so that git pull always rebases by default across your team's machines?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Protected Branches and Required Checks](<../../../../TOOLS/Git_and_GitHub/Git_Core/Protected_Branches_and_Required_Checks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that git pull always rebase by default, we need to configure Git settings on each developer’s machine to use the "rebase" strategy when pulling changes from a remote repository. This setup promotes cleaner commit histories and avoids unnecessary merge commits.

#### Triage

The question is essentially asking how to set up a global configuration in Git so that git pull defaults to performing a rebase operation instead of a merge. We'll need to use the .gitconfig file, which allows us to specify default behavior for various Git commands across all repositories on a machine.

#### Assumptions
Each developer has access to modify their local .gitconfig files.
The team is using Git repositories hosted on GitHub or another platform that supports git pull.
Developers are working in a Unix-like environment (e.g., Linux, macOS) where .gitconfig is configured for the shell.

#### Solution
Step-by-Step Solution:

Open or Create the Global Configuration File: The global configuration file, located at ~/.gitconfig, stores settings that apply to all Git repositories on a given machine. We need to modify this file to set up our desired behavior for git pull.

Edit the .gitconfig File: Use your preferred text editor (e.g., nano, vim) to open or edit the global .gitconfig file.

Add the Rebase Command: Add the following line under the [pull] section of the .gitconfig file:

INI
[pull]
    rebase = true

This tells Git that for every git pull, it should perform a rebase instead of a merge.

Save and Close: Save the changes to your .gitconfig file and close the editor.

Verify the Configuration: Run git config --global --list to confirm that the setting has been applied correctly:

SHELL
git config --global --list | grep pull.rebase=true

#### Closing Summary
+ Common Mistakes/Follow-Ups:

By following these steps, you ensure that every git pull operation on all of your team's machines will perform a rebase. This setup helps maintain a clean project history and ensures that merge commits are avoided.

#### Common Mistakes
to Avoid:

Not Using the Global Configuration: If you do not use the global configuration file, each repository would require individual settings.
Incorrect Path for .gitconfig File: Ensure that the .gitconfig file is stored in ~/.gitconfig (Unix-like systems) or %USERPROFILE%\.gitconfig (Windows).

#### Follow-Up
Questions:

How does Git handle conflicts during a rebase?
Are there any situations where it might be better to use a merge instead of rebasing?

**Code:**

```
INI
Copy
[pull]
    rebase = true
```

