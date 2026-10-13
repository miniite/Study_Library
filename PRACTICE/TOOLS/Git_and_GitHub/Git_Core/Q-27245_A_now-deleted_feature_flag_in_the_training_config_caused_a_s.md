---
type: "practice"
difficulty: "Deep"
---

# A now-deleted feature flag in the training config caused a stale-encoder bug and you must find when it was introduced and removed across tens of thousands of commits. Explain git log -S versus -G and why regex vs literal matters for this hunt.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Searching History](<../../../../TOOLS/Git_and_GitHub/Git_Core/Searching_History.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of finding a now-deleted feature flag in the training config that caused a stale-encoder bug, we need to leverage Git's powerful search capabilities across tens of thousands of commits. The key tools here are git log -S and git log -G, which serve different purposes based on whether you're searching for exact matches or regular expression patterns.

The essence of this question is about understanding the nuances between using -S versus -G in Git, particularly when dealing with literal strings versus regexes. Here’s a step-by-step solution from first principles:

Understand the Need: We need to pinpoint the introduction and removal of a specific feature flag (e.g., STALE_ENCODER_FLAG) across thousands of commits.
Git Log -S: This command searches for the addition or deletion of a literal string in the commit history. It’s useful when you know exactly what text is involved, but it doesn’t support regex patterns.
Git Log -G: This command uses regular expressions to search for changes that match a specific pattern. It's more flexible than git log -S and can be powerful in complex searches.

#### Triage
The interviewer is probing your understanding of Git’s advanced searching capabilities, particularly the differences between literal string matching (git log -S) and regex-based pattern matching (git log -G).
You need to explain why using a literal match versus a regex matters for this specific task.

#### Assumptions
We have a large codebase with tens of thousands of commits, and we are looking for the introduction or removal of a feature flag.

#### First-Principles Solution
Step 1: Identify the Feature Flag

Let's assume the feature flag is STALE_ENCODER_FLAG.

Using git log -S:

SH
git log -S 'STALE_ENCODER_FLAG'

This command will show commits that added or deleted exactly the string STALE_ENCODER_FLAG. If the exact flag is used consistently, this could be straightforward.

Using git log -G with a regex:

SH
git log -G 'STALE_ENCODER_FLAG'

This command allows you to use a regular expression to search for patterns. For example, if the flag was introduced as part of another phrase or with slight variations, this would be more flexible.

Step 2: Determine the Introduction and Removal Dates

Once we have the commits from both commands, we can look at the commit messages and dates to determine when the feature flag was first introduced and last used. For example:

SH
git show <commit_hash>

This will display the commit details, including the commit message which often contains context about changes.

Step 3: Analyze Commit Messages and Surrounding Code Changes

Examine the context around the introduction and removal of the feature flag to understand its impact. For example:

Look for references in other files or comments.
Check if there are related pull requests or issues that mention the feature flag.
Verify if the flag was part of a larger refactoring effort.

#### Closing Summary

In summary, git log -S is ideal when you know exactly what text to search for, while git log -G allows more flexibility with regular expressions. The choice between these depends on the specific context and the nature of your search.

#### Common Mistakes/Follow-Ups
Mistake: Using git log -G without a regex pattern might return false positives if not carefully crafted.

#### Follow-Up
Discuss potential performance implications when using regexes in large repositories, as they can be slower than literal string searches.

**Code:**

```
SH
Copy
# Example commands to use Git for the search
git log -S 'STALE_ENCODER_FLAG'
git log -G 'STALE_ENCODER_FLAG'
```

