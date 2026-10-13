---
type: "practice"
difficulty: "Intermediate"
---

# How would you use git log with a filter to find every commit where you changed the retrieval or embedding code over the last month?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Searching History](<../../../../TOOLS/Git_and_GitHub/Git_Core/Searching_History.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To find every commit in the last month that involved changes to retrieval or embedding code, we can leverage git log with specific filters and regular expressions. The goal is to identify commits where the file paths match those related to retrieval or embedding logic.

#### Triage

The question asks for a way to filter git logs based on specific criteria (file paths) over a time range (last month). We will use git log combined with shell commands like grep and awk to accomplish this.

#### First-Principles Solution

Identify the Date Range:

Use the --since option in git log to specify the start of the date range.

Filter by File Paths:

Use a regular expression with grep or awk to match file paths related to retrieval and embedding code.

Combine Commands:

Chain git log, grep, and any necessary post-processing steps (like awk) to filter the commits accordingly.
Step-by-Step Solution:
1. Determine the Date Range

To find commits in the last month, we can use a command like:

SH
LAST_MONTH=$(date -d "last month" +%Y-%m-%d)
git log --since="$LAST_MONTH"
2. Filter by File Paths

Assuming common file extensions for retrieval and embedding code might include .py, .js, or .rb (Python, JavaScript, Ruby), we can use a regular expression to match these.

The command becomes:

SH
git log --since="$LAST_MONTH" -- $(grep -rlE 'retrieval|embedding' . | awk -F: '{print $1}')
--since="$LAST_MONTH" specifies the date range.
$(grep -rlE 'retrieval|embedding' .) finds all file paths containing "retrieval" or "embedding".
awk -F: '{print $1}' strips out the line numbers and just keeps the file path.

#### Closing Summary

This command effectively filters git logs to show only those commits in the last month that involve changes to retrieval or embedding code. The key steps are setting up the date range, identifying relevant files using a regular expression, and chaining these into git log for filtering.

#### Common Mistakes/Follow-Ups
Common Mistake: Failing to account for file paths with different extensions (e.g., .py, .js). This can be addressed by including common extension filters or adjusting the regex pattern.

#### Follow-Up
If embedding and retrieval code are part of a larger project, it might be worth considering additional directories or more specific patterns to ensure relevant changes are captured.

**Code:**

```
SH
Copy
LAST_MONTH=$(date -d "last month" +%Y-%m-%d)
git log --since="$LAST_MONTH" -- $(grep -rlE 'retrieval|embedding' . | awk -F: '{print $1}')
```

**Example input / output:**

```text
$ git log --since="2023-09-01" -- $(grep -rlE 'retrieval|embedding' . | awk -F: '{print $1}')
commit 123456789abcdef
Author: John Doe <johndoe@example.com>
Date:   Mon Sep 4 10:00:00 2023 +0200

    Updated retrieval logic in retrieval.py
```

**Flow:**

1. yes
2. no
3. Start
4. Is today >= last month?
5. Calculate LAST_MONTH date
6. Run git log with specifiedoptions
7. Display filtered commits

