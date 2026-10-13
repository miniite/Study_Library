---
type: "practice"
difficulty: "Intermediate"
---

# How would you use git log or git blame to find when a data preprocessing bug that skewed your features was introduced?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Blame and Ignore Revisions](<../../../../TOOLS/Git_and_GitHub/Git_Core/Blame_and_Ignore_Revisions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this interview question, we need to understand how Git commands like git log and git blame can be leveraged to trace back to the commit where a data preprocessing bug that skewed features was introduced. We'll start by summarizing these commands and their usage, then move on to a detailed explanation of the steps involved.

Firstly, git log is used to display changes in commits across a repository, while git blame provides information about which line of code was last modified and who made that change, along with the commit ID. These tools are essential for version control and debugging in software development projects, including machine learning pipelines where data preprocessing is critical.

Let's break down the steps to find when a bug that skewed features was introduced:

Identify Suspect Files: First, determine which files contain the data preprocessing logic. For instance, if you suspect the data_preprocessing.py file might be responsible for the issue.
Use Git Log to Find Changes in the File: Run git log -- <file> (where <file> is the name of your suspect file). This command will show a list of all commits that have modified this particular file, along with commit hashes and authors.
Analyze Commits Using Git Blame: For each significant change identified by git log, use git blame <commit-hash> -- <file> to pinpoint the exact line of code where changes were made. This command provides a detailed view of who changed what lines in that specific commit.
Review Changes for the Bug: Once you have narrowed down the list of commits, manually review or use tools like git diff (e.g., git diff <commit-hash>^! -- <file> to compare changes) to find where and when the bug was introduced.

By systematically using these Git commands, we can trace back through history to identify the exact commit responsible for introducing a bug that skewed our features.

**Comparison:**

```
Commit Hash	Author	Date	File
c974e56	Alice Smith	2023-10-05	data_preprocessing.py
```

**Code:**

```bash
# Example usage of git log and git blame

# Step 1: Find commits that modified data_preprocessing.py
git log -- data_preprocessing.py

# Step 2: Inspect specific lines in those commits using git blame
git blame c974e56 -- data_preprocessing.py
```

**Example input / output:**

```text
# Example git log output
commit c974e56 (HEAD -> main, origin/main)
Author: Alice Smith <alice.smith@example.com>
Date:   Tue Oct 5 14:00:00 2023 +0200

    Added feature scaling in data_preprocessing.py

# Example git blame output
c974e56 (Alice Smith) df['feature'] = df['raw_data'].apply(lambda x: x / 10) # line 10
```

**Data:**

```json
{
    "commit": {
        "hash": "c974e56",
        "author": "Alice Smith",
        "date": "2023-10-05T14:00:00Z"
    },
    "file": "data_preprocessing.py",
    "lines": [
        {"line_number": 10, "content": "df['feature'] = df['raw_data'].apply(lambda x: x / 10)", "last_modified_by": "Bob Johnson", "date": "2023-10-06T10:00:00Z"},
        {"line_number": 15, "content": "# df['feature'] = df['raw_data'].apply(lambda x: np.log(x + 1))", "last_modified_by": "Alice Smith", "date": "2023-10-07T16:00:00Z"}
    ]
}
```

**Flow:**

1. Identify Suspect Files
2. Run 'git log -- <file>'
3. Analyze commits using `gitblame`
4. Review changes for the bug
5. Identify the commitresponsible for introducing thebug

