---
type: "practice"
difficulty: "Intermediate"
---

# A mass reformat commit poisons git blame across your service layer, hiding the real author of a concurrency bug. Set up .git-blame-ignore-revs and explain the limits of that mechanism during a live incident.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Blame and Ignore Revisions](<../../../../TOOLS/Git_and_GitHub/Git_Core/Blame_and_Ignore_Revisions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

When dealing with a mass reformat commit in Git, it's essential to understand how git blame works and why using .git-blame-ignore-revs might not be sufficient to fully resolve issues related to tracking the real author of changes. This mechanism can help mitigate some visibility problems but has limitations that must be considered during an incident.

#### Triaging the Question

The question revolves around a specific Git issue—reformatting code in one massive commit, which affects git blame output and obscures the original authorship of changes. We need to set up .git-blame-ignore-revs to mitigate this problem but must also be aware of its limitations.

#### First Principles Solution

Understanding Git Blame:

git blame traces each line in a file back to the commit that last modified it.
It's used for understanding who made changes and when, which is crucial during code review and debugging.

Mass Reformat Commit Issue:

A single commit with extensive reformatting can overwrite git blame output, making it difficult to trace changes accurately.

Setting Up .git-blame-ignore-revs:

The .git-blame-ignore-revs file allows you to exclude certain revisions from being considered in the blame output.
This is useful when dealing with large refactors or reformatting that might obscure authorship.

#### Implementation
Steps:

Identify the problematic commit hash.
Create a .git-blame-ignore-revs file and add the problematic commit hash to it.
Commit this change to ensure Git recognizes the exclusion.

Limitations of .git-blame-ignore-revs:

It only works for blame output from that specific commit onwards, not for historical blame outputs.
Other collaborators might still see the old git blame information in their local repositories until they fetch and update the latest changes.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

Correctness and Visibility: While .git-blame-ignore-revs helps improve visibility, it doesn't completely solve all issues related to tracking authorship.

#### Common Mistakes

Over-relying on git blame as the sole method for identifying changes.
Not considering other Git tools (e.g., git log, git show) that might provide more context.

#### Follow-Up
Questions:
How would you handle a scenario where multiple reformat commits occur within a short timeframe?
Can you discuss how version control policies can mitigate such issues in the long term?

**Comparison:**

```
Commit Hash	Date	Author	Message
d8e54c7	2023-10-01	Alice	Massive reformat of service layer code.
c4b9a8d	2023-10-02	Bob	Bug fix in service logic.
```

**Code:**

```bash
# Example of creating and committing .git-blame-ignore-revs
echo "d8e54c7" > .git-blame-ignore-revs  # Replace with actual commit hash
git add .git-blame-ignore-revs
git commit -m "Exclude problematic reformat commit from blame"
```

**Example input / output:**

```

Input:

Problematic commit hash: d8e54c7

Output:

BASH
Copy
$ git blame <file_path> --ignore-revs-file=.git-blame-ignore-revs

This command will ignore the problematic commit and provide accurate blame information.
```

**Data:**

```json
{
  "commit_hash": "d8e54c7",
  "repository_name": "example-service-layer",
  "branch_name": "master"
}
```

**Flow:**

1. Identify Problematic Commit
2. Set up .git-blame-ignore-revs
3. Commit Exclusion
4. Run 'git blame' with--ignore-revs-file option

