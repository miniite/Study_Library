---
type: "practice"
difficulty: "Intermediate"
---

# A now-removed feature flag in a stored procedure caused a stale-config query bug and you must find when it was introduced and later removed across tens of thousands of commits. Explain git log -S versus -G and why literal versus regex matters for this hunt.

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Searching History](<../../../../TOOLS/Git_and_GitHub/Git_Core/Searching_History.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need to use Git's search capabilities within commit messages and file content to find when a specific feature flag was introduced and then removed. Let’s break down the tools at our disposal and why certain options are more suitable for this task.

TL;DR

We will use git log -S to search through the commit history of a repository for specific strings (e.g., SQL code snippets) that match our query, as it provides better control over the regex patterns. In contrast, git log -G is used for searching file contents with regular expressions directly, but in this case, using -S is more appropriate because we need to find a pattern across commit messages and file content.

#### Triaging the Question

The question requires us to pinpoint when a specific feature flag was introduced and later removed. We are working with tens of thousands of commits, so efficiency is crucial. The key tools here are git log -S and git log -G, each with their own strengths and use cases.

#### Assumptions
and Context
Repository Size: Tens of thousands of commits.
Search Scope: Need to find both commit messages and file contents related to the feature flag.
Feature Flag: The flag is present in SQL code, possibly as a string or variable name.
Git Version: Assuming modern Git with these commands available.

#### First-Principles Solution
Step 1: Understanding git log -S

git log -S allows us to search for a specific string (e.g., "feature_flag") in commit messages and file contents:

BASH
git log -S 'feature_flag'

This command searches the entire history of the repository, looking for commits where this exact string was added or removed. It’s particularly useful when you need to understand how the flag's presence has evolved over time.

Step 2: Understanding git log -G

git log -G is used to search file contents with regular expressions:

BASH
git log -G 'feature_flag'

This command searches for lines in files that match a given regular expression. It’s more suited for searching patterns within the code itself, not just commit messages.

Why Literal vs Regex Matters

Literal (-S): When using -S, we are looking for exact matches of the string "feature_flag". This is useful when you know the exact term and want to ensure no other similar terms interfere.

BASH
git log -S 'feature_flag'

Regex (-G): When using -G, we can use regular expressions to match patterns. However, this might be overkill for our specific task of finding exact string matches.

Practical Example

Suppose we want to find when the feature flag my_feature_flag was introduced and removed:

BASH
# Find commits introducing or removing "feature_flag" exactly
git log -S 'feature_flag'

# If using a regular expression, it might look like:
git log -G '\bfeature_flag\b'

#### Closing Summary

In summary, for this task of finding when a specific feature flag was introduced and later removed across tens of thousands of commits, we should use git log -S to search both commit messages and file contents. This command provides the precision needed without the complexity introduced by regular expressions.

#### Common Mistakes
and Follow-Ups
Mistake: Using -G instead of -S when exact string matches are required.

#### Follow-Up
Discussing scenarios where git grep, git blame, or other tools might be useful for further investigation after identifying the initial commit range from git log -S.

