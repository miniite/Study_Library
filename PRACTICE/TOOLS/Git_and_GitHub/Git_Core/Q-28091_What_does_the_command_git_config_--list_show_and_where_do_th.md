---
type: "practice"
difficulty: "Basic"
---

# What does the command git config --list show and where do those settings actually live on disk?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Reviewing Changes with Diff](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reviewing_Changes_with_Diff.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand what git config --list shows, we first need to grasp the concept of Git configuration. Git has a hierarchical structure for storing configuration settings, ranging from system-wide to user-specific and repository-specific levels. The command git config --list lists all these configurations in a readable format.

#### Triaging the Question

The question is asking about the output of git config --list and where this data is stored on disk. To answer this, we need to delve into how Git organizes its configuration files.

#### First-Principles Solution

Git stores configuration settings in three main places:

System-wide Configuration: Located at /etc/gitconfig (Unix/Linux) or C:\ProgramData\Git\config (Windows).
User-specific Configuration: Found in the user's home directory, typically at ~/.gitconfig.
Repository-specific Configuration: Stored within each repository under .git/config.

When you run git config --list, Git merges these three levels of configurations. The system-wide settings are read first, then the user-specific ones, and finally the repository-specific settings. If there are conflicts or duplicate keys across these files, the values from the higher-priority level override those from lower priority.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, git config --list shows a combined view of all your Git configuration options, originating from three primary locations: system-wide, user-specific, and repository-specific settings. The command does not list these files directly; instead, it reads the configuration from these locations.

A common mistake is assuming that running git config --list will show the actual file contents rather than the values those files define. To see the exact content of a specific configuration file, you can use commands like cat ~/.gitconfig or less /etc/gitconfig.

Self-Check

This answer covers the core concept and potential misunderstandings about Git's configuration system.

**Code:**

```bash
# Example command to list configurations (in practice, it shows parsed output)
git config --list
```

